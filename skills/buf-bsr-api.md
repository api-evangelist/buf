---
name: buf-bsr-api
description: >-
  Call the Buf Schema Registry (BSR) public API over Connect/gRPC — look up modules
  and owners, list commits and labels, download module contents, and fetch a compiled
  FileDescriptorSet. Use when an agent needs BSR data programmatically instead of
  through the buf CLI or the web UI.
api: buf:buf-schema-registry
base_url: https://buf.build
transport: Connect (HTTP/1.1 + HTTP/2, JSON bodies), gRPC, gRPC-Web
operations:
  - buf.registry.owner.v1.OwnerService/GetOwners
  - buf.registry.module.v1.ModuleService/GetModules
  - buf.registry.module.v1.ModuleService/ListModules
  - buf.registry.module.v1.CommitService/ListCommits
  - buf.registry.module.v1.LabelService/ListLabels
  - buf.registry.module.v1.DownloadService/Download
  - buf.registry.module.v1.GraphService/GetGraph
  - buf.registry.module.v1.FileDescriptorSetService/GetFileDescriptorSet
  - buf.reflect.v1beta1.FileDescriptorSetService/GetFileDescriptorSet
generated: '2026-09-13'
method: generated
source: >-
  grpc/buf/registry/**/*.proto (verbatim from github.com/bufbuild/registry-proto) and
  https://buf.build/docs/bsr/apis/api-access/
---

# Calling the Buf Schema Registry API

Every BSR RPC is mounted at the **root** of the BSR host — a gRPC compatibility
requirement — so the URL is always:

```
https://<bsr-host>/<fully.qualified.ServiceName>/<MethodName>
```

`<bsr-host>` is `buf.build` for the public BSR, `<org>.buf.dev` for a Pro instance,
or your own domain for Enterprise. Never invent a `/v1/...` REST path; the BSR has
none.

## 1. Authenticate

Send a BSR API token as a bearer token:

```
Authorization: Bearer ${BUF_TOKEN}
```

Create the token in account settings, or run `buf registry login` and read it from
`$HOME/.netrc`. Public modules on `buf.build` accept unauthenticated reads; private
instances generally require a token on every call. See
`authentication/buf-authentication.yml`.

## 2. Resolve what you were given

Users hand you a module reference like `buf.build/connectrpc/eliza`. Split it into
owner (`connectrpc`) and module (`eliza`), then:

- `buf.registry.owner.v1.OwnerService/GetOwners` — is the owner a user or an organization?
- `buf.registry.module.v1.ModuleService/GetModules` — module id, visibility, state, default label.
- `buf.registry.module.v1.ResourceService/GetResources` — one call that resolves a module,
  label, or commit reference when you do not yet know which one you hold.

## 3. Read history

- `buf.registry.module.v1.CommitService/ListCommits` — commits for a module.
- `buf.registry.module.v1.LabelService/ListLabels` and `/ListLabelHistory` — labels and
  what each one pointed at over time.

All list RPCs page the same way: send `page_size` (max **250**) and `page_token`
(max 4096 bytes), read `next_page_token` from the response, stop when it is empty.
Most list requests also accept an `order` enum defaulting to `ORDER_CREATE_TIME_DESC`.

## 4. Get the schema itself

- `buf.registry.module.v1.DownloadService/Download` — the `.proto` file contents of a commit.
- `buf.registry.module.v1.FileDescriptorSetService/GetFileDescriptorSet` — the compiled
  descriptor set for a module in the registry API.
- `buf.reflect.v1beta1.FileDescriptorSetService/GetFileDescriptorSet` — the separate
  beta reflect module, which is the one Buf's own docs use in their curl example:

```bash
curl https://buf.build/buf.reflect.v1beta1.FileDescriptorSetService/GetFileDescriptorSet \
  -H "Authorization: Bearer ${BUF_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"module": "buf.build/connectrpc/eliza"}'
```

`buf.registry.module.v1.GraphService/GetGraph` returns the dependency graph when you
need the transitive closure rather than one module.

## 5. Writes, and taking them back

Write RPCs come in plural, atomic batches — `CreateModules`, `UpdateModules`,
`DeleteModules`, `CreateOrUpdateLabels`, `ArchiveLabels` — and each is all-or-nothing:
either every element in the request succeeds or the call returns an error.

Every one of them declares `option idempotency_level = IDEMPOTENT` in the contract,
so a retry after a timeout is safe. The four `Upload` RPCs do **not** declare it;
treat an interrupted upload as unknown and reconcile with `ListCommits` before
retrying. See `conventions/buf-conventions.yml`.

Reversal paths that actually exist:

| Action | Reversal |
|---|---|
| `ArchiveLabels` | `UnarchiveLabels` |
| `buf registry module deprecate` | `buf registry module undeprecate` |
| `DeleteModules` / `DeleteOrganizations` / `DeleteUsers` | none — destructive, no documented undo window |

Buf publishes no restore window for deletes. Do not promise a user one.

## 6. Handle failures

Errors come back as Connect error codes. On `resource_exhausted` you have hit a rate
limit: the response carries HTTP 429, `X-RateLimit-Remaining: 0` and `Retry-After`.
Honor `Retry-After` rather than retrying sooner. Buckets are 30 req/sec sustained
(burst 60) for the general API and 1 req/sec (burst 2) for
`FileDescriptorSetService`. See `rate-limits/buf-rate-limits.yml` and
`errors/buf-problem-types.yml`.

## 7. When an MCP client is available

The same API is exposed as an MCP server at `https://buf.build/mcp` (OAuth2, scope
`mcp`, or a bearer BUF_TOKEN header for CI). Prefer it when the agent host supports
remote MCP — the tools are the same v1 services listed above.

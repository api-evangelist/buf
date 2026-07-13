---
title: "Introducing hyperpb: 10x faster dynamic Protobuf parsing that’s even 3x faster than generated code"
url: "https://buf.build/blog/hyperpb/"
date: "2025-07-16"
feed_url: "https://buf.build/blog/rss.xml"
---
Today we’re announcing public availability of hyperpb, a fully-dynamic Protobuf parser that is 10x faster than dynamicpb, the standard Go solution for dynamic Protobuf. In fact, it’s so efficient that it’s 3x faster than parsing with generated code! It also matches or beats vtprotobuf’s generated code at almost every benchmark, without skimping on correctness.

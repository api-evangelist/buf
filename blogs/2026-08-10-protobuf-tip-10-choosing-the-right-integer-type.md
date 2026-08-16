---
title: "Protobuf Tip #10: Choosing the right integer type"
url: "https://buf.build/blog/choosing-the-right-protobuf-integer-type"
date: "2026-08-10"
feed_url: "https://buf.build/blog/rss.xml"
---
Protobuf has ten integer types. Use int64. In Go, even the slow cases decode a thousand integers in a few microseconds, so the choice almost never matters.

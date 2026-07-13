---
title: "Oneofs are a disaster. Protovalidate has fixed them."
url: "https://buf.build/blog/fixing-oneofs/"
date: "2025-06-13"
feed_url: "https://buf.build/blog/rss.xml"
---
Instead of using oneofs, you can now use the new (buf.validate.message).oneof Protovalidate annotation. As long as you're validating your messages with Protovalidate, (buf.validate.message).oneof does exactly what you'd expect, with none of the pain.

---
title: "Tip of the week #9: Some numbers are more equal than others"
url: "https://buf.build/blog/totw-9-some-numbers-are-more-equal-than-others/"
date: "2025-06-17"
feed_url: "https://buf.build/blog/rss.xml"
---
The first 15 field numbers are special: most runtimes will decode them much faster than the other field numbers. When designing a message type for decoding performance, it’s good to use these field numbers on fields that are almost always present.

---
layout: post
title:  "Making DecodableError more readable"
date:   2024-08-01 12:00:00 +0000
categories: swift ios macos
excerpt_separator: <!--more-->
---

In an ideal world, the backend API that your app uses is well documented and resolutely sticks to the specification. In the real world there will likely be circumstances where that isn't true, maybe because an API is still being developed. While the `DecodingError` returned by `JSONDecoder.decode()` comprehensively describes the problem, it's not the easiest to read. Maybe there's something we can do about that...

<!--more-->


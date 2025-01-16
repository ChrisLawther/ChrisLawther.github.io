---
layout: post
title:  "Automated reminders to revisit old code"
date:   2025-01-16 12:00:00 +0000
categories: swift ios macos
excerpt_separator: <!--more-->
---

Sometimes you may find yourself in a situation in which you are aware of tooling changes that will offer a better solution to something you're working on, but for whatever reason they're not available to you right now. (The most likely reason being an Xcode version that isn't yet supported for AppStore releases)

There's a simple annotation which you can add to your code so that Xcode will remind you to revisit that piece of work when you do upgrade.

<!--more-->

(NOTE: This isn't anything novel or new, but I'm posting it here to make it easier for *me* to find!)

`@available` is our friend for this one. Annotate a function that you *would* write differently, if only the latest and greatest toolchain improvements were available, as follows:

{% highlight swift %}
@available(swift, deprecated: 6.1,
           message: "Switching to the new XYZ implementation will make this better")
function doSomething() { ... }
{% endhighlight %}

(At time of writing 6.0 is current, so we'll start seeing that deprecation warning with the next `.1` update)

---
layout: post
title:  "A gotcha with default implementations and a workaround to default args"
date:   2024-06-01 12:00:00 +0100
categories: swift ios macos
excerpt_separator: <!--more-->
---

*Generally* it's ok to do ...

{% highlight swift %}
protocol DoStuff {

}
{% endhighlight %}

Except when it isn't...

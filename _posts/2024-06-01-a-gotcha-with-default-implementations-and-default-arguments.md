---
# layout: post
title:  "A gotcha with default implementations and a workaround to default args"
date:   2024-08-06 21:55:52 +0100
categories: swift ios macos
---

*Generally* it's ok to do ...

{% highlight swift %}
protocol DoStuff {

}
{% endhighlight %}

Except when it isn't

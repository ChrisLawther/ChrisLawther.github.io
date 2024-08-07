---
# layout: post
title:  "Configuring your app at launch"
date:   2024-08-06 21:55:52 +0100
categories: ios swift testing testability
---

Two (3?) reasons:

 1. You want your tests to have control over the data the app sees
 1. You want ... (dev-mode?)

 Makes use of two features of `UserDefaults`: default values and command-line overrides.

### Default values

UserDefaults has an API for registering default ephemeral values (i.e. they can be read, but they're never persisted). These values will be returned by `valueForKey:` as long as no other value has ever been set. This means you can define and apply sensible defaults for your app in a single place, rather than at point of use.

### Command line overrides

UserDefaults values can be temporarily over-written (or maybe "obscured" is a better description) by values passed on the command line, something only tests and XCode schemes can do (in other words, it's a mechanism only available to the developer).

For example, passing the command line arguments `-foo` and `bar` would cause `UserDefaults.standard.string(forKey: "foo")` to return `"bar"` rather than any pre-existing value, as long as the app hadn't written another value to it post-launch.

## Setting values

### In the app

Early on in the app's life-cycle (i.e. before anything *reads* `UserDefaults`), register your default values, passing a dictionary of `[String: Any]`:

{% highlight swift %}
UserDefaults.standard.register(defaults: [
	"foo": "bar",
	"someBoolean": true,
	"baseURL": "https://myapp.com/api"
])
{% endhighlight %}

### From a Scheme

In Product → Scheme → Edit Scheme (`⌘ <`, or more practically, `⌘ Shift ,`), select "Run" from the sidebar and then the "Arguments" tab. Add two arguments to "Arguments Passed on Launch" for each variable you want to override. Set the value for the first in each pair to the `UserDefaults` key prefixed with a single `-` (i.e. to override `baseURL`, the argument would be `-baseURL`) and set the second value to the desired value (e.g. "https://staging.myapp.com/api")

### From a UITest

You might want a UITest to be able to control the data the app is operating on, for example to confirm empty or error-state behaviour. This might mean pointing the app at some internal environment over which you have control. Alternatively, the app could be configured to talk to the machine running the tests. I've used the lightweight http server [swifter](https://github.com/httpswift/swifter) for this in the past. Incidentally, this also gives you the opportunity to record and inspect the requests your app makes, although you may already have that covered in unit tests.

{% highlight swift %}
let app = XCUIApplication()
app.launchArguments = [
	"-foo", "bar",
	"-someBoolean", true,
	"-baseURL", "http://localhost:3000"
]
app.launch()
{% endhighlight %}


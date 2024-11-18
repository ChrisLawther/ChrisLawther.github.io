---
# layout: post
title:  "Configuring your app at launch"
date:   2024-08-06 21:55:52 +0100
categories: ios swift testing testability
---

For a few reasons, we sometimes want to be able to control where our app gets its data from:

 1. Production for public users
 1. A staging environment to allow stake-holders to see how the app handles changes before they go to production
 1. Curated data for screenshot generation
 1. Tests. In the interests of speed, predictability and reliability, of course your unit tests shouldn't be using any network, but you may have tests high up the pyramid that do want to use a real network, just not the default production environment.

Ideally we'd like to achieve that in a single build of the app ("test what you ship"), with as little added complexity as possible. No more `#ifdef DEBUG`!

Two features of `UserDefaults` (one of them not widely known) help us to achieve that goal:

### Default values

UserDefaults has an API for registering default ephemeral values (i.e. they can be read, but they're never persisted). These values will be returned by `valueForKey:` as long as no other value has ever been set. This means you can define and apply sensible defaults for your app in a single place, rather than at point of use.

### Command line overrides

UserDefaults values can be temporarily over-written (or maybe "obscured" is a better description) by values passed on the command line, something only tests and XCode schemes can do (in other words, it's a mechanism only available to the developer).

For example, passing the command line arguments `-foo` and `bar` would cause `UserDefaults.standard.string(forKey: "foo")` to return `"bar"` rather than any pre-existing value. If any logic in the app _changes_ the value post-launch, that new value would be seen instead.

## Setting values

### In the app

For normal operation, so that everything works as intended for Joe Public, we need to ensure that all of the required default values exist.

Early on in the app's life-cycle (i.e. before anything *reads* `UserDefaults`), register your default values, passing a dictionary of `[String: Any]`:

{% highlight swift %}
UserDefaults.standard.register(defaults: [
	"foo": "bar",
	"someBoolean": true,
	"baseURL": "https://myapp.com/api"
])
{% endhighlight %}

### From a Scheme

For day-to-day development, we may wish (or be required) to work against something other than the production environment.

In Product → Scheme → Edit Scheme (`⌘ <`, which on my keyboard at least is achieved by `⌘ Shift ,`), select "Run" from the sidebar and then the "Arguments" tab. Add two arguments to "Arguments Passed on Launch" for each variable you want to override. Set the value for the first in each pair to the `UserDefaults` key prefixed with a single `-` and set the second value to the desired value.

For example, to set `baseURL` to `"https://staging.myapp.com/api"` you would need these as two consecutive arguments:

 * `-baseURL`
 * `https://staging.myapp.com/api`

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

(Since screenshot generation is generally just a special case of UI test, the same applies)

## Next steps

That has the original points 1, 3 and 4 covered. For point #2, we'll need another way to influence the app if we want it to target a different environment for internal stake-holders, or perhaps apply some other configuration change, all while sticking to a single build. Perhaps in a future article...

## Final thoughts

I've yet to encounter anybody using this approach. Maybe it's deeply flawed, or there's a a better way?
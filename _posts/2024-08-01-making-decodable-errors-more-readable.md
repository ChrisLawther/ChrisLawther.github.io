---
layout: post
title:  "Making DecodableError more readable"
date:   2024-08-01 12:00:00 +0000
categories: swift ios macos
excerpt_separator: <!--more-->
---

In an ideal world, the backend API that your app uses is well documented and resolutely sticks to the specification. In the real world there will likely be circumstances where that isn't true, maybe because an API is still being developed. While the `DecodingError` returned by `JSONDecoder().decode(:from:)` comprehensively describes the problem, it's not the easiest to read. Maybe there's something we can do about that...

<!--more-->

There are currently four variants of [`DecodingError`][1], encoded as an enum with associated values. All have a `DecodingError.Context` describing exactly where the error was. The `.path` property is an array of `any CodingKey` which have two properties of interest to us:


{% highlight swift %}
/// A type that can be used as a key for encoding and decoding.
public protocol CodingKey {   
    /// The string to use in a named collection (e.g. a string-keyed dictionary).
    var stringValue: String { get }
    
    /// The value to use in an integer-indexed collection (e.g. an int-keyed
    /// dictionary).
    var intValue: Int? { get }
}
{% endhighlight %}

As an example, given these models:

{% highlight swift %}
struct Model: Decodable {
    let children: [Child]
}

struct Child: Decodable {
    let id: Int
}
{% endhighlight %}

... when asked to decode this mis-matched JSON

{% highlight json %}
{
    "children": [
        {"id": 1},
        {"id": "two"},
        {"id": 3},
    ]
}
{% endhighlight %}

... the thrown `DecodingError` would have three path elements describing the location of the problem:

{% highlight swift %}
[
    CodingKeys(stringValue: "children", intValue: nil),
    CodingKey(stringValue: "Index 1", intValue: 1),
    CodingKeys(stringValue: "id", intValue: nil)
]
{% endhighlight %}

In other words, within "children", the 2nd element has an error in the representation of its `id` property. A more concise way of expressing that could be to reduce it to the string `".children[1].id"`, as implemented by:

{% highlight swift %}
func humanReadablePath(from path: [CodingKey]) -> String {
    guard !path.isEmpty else { return "." }
    
    return path.map { key -> String in
        if let index = key.intValue { return "[\(index)]" }
        return ".\(key.stringValue)"
    }
    .joined()
}
{% endhighlight %}

The more deeply nested the error, the bigger the advantage of this alternative representation becomes.

`FriendlyDecodableError` preserves the four variations of `DecodingError`, but presents the location of the error in our new easier-to-read format:

{% highlight swift %}
public enum FriendlyDecodableError: Error {
    public static func from(_ decodingError: DecodingError) -> FriendlyDecodableError {
        switch decodingError {
        case let .valueNotFound(dataType, context):
            return .missingValue(type: String(describing: dataType),
                                 path: humanReadablePath(from: context.codingPath))
            
        case let .keyNotFound(key, context):
            return .keyNotFound(key: key.stringValue,
                                path: humanReadablePath(from: context.codingPath))
            
        case .dataCorrupted:
            return .corruptedData
            
        case let .typeMismatch(dataType, context):
            return .typeMismatch(expected: String(describing: dataType),
                                 path: humanReadablePath(from: context.codingPath))
            
        @unknown default:
            assertionFailure("DecodingError gained a new case that we're not handling yet")
            return other
        }
    }
    
    /// The model has a required property, but the data has a `null` value
    /// - Note: Some types (e.g. `Bool`) return a `typeMismatch` instead.
    /// - Parameters:
    ///   - type: The expected type of the missing value
    ///   - path: The path to the property in the model
    case missingValue(type: String, path: String)
    
    /// A required property of the model is not present in the data
    /// - Parameters:
    ///   - key: The name of the property
    ///   - path: The path to the property in the model
    case keyNotFound(key: String, path: String)
    
    /// The data could not be parsed as JSON
    case corruptedData
    
    /// The JSON type does not match the model's requirements
    /// - Parameters:
    ///   - expected: The type of the property in the model
    ///   - path: The path to the property in the model
    case typeMismatch(expected: String, path: String)
    
    /// An unknown error occurred
    case other
}
{% endhighlight %}

For a little added convenience, we could also extend `DecodingError` so that we can ask it directly for the friendly representation:

{% highlight swift %}
public extension DecodingError {
    func friendlyDecodingError() -> FriendlyDecodableError {
        FriendlyDecodableError.from(self)
    }
}
{% endhighlight %}



  [1]: https://developer.apple.com/documentation/swift/decodingerror
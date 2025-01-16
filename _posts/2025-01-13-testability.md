---
layout: post
title:  "Testability, the concept and the package"
date:   2025-01-13 12:00:00 +0000
categories: swift ios macos
excerpt_separator: <!--more-->
---

When working in discrete chunks of code in small SPM packages, a recurring pattern is the desire to make code using some aspect of `Foundation` testable. Rather than implement the same pattern over and over again, I've put those I've encountered recently into [a package][1], with more to come, probably.

<!--more-->

At time of writing this covers data fetching, simple file downloading, file movement and file attribute inspection and modification.

Here's a reasonably realistic example of when all of these may come together. Let's say we're making an app to keep a local archive of all episodes of a favourite podcast. It's going to:

 * fetch some JSON or XML data describing the current episodes
 * download zero or more of those episodes
 * move the downloaded file to a permanent location
 * set the `created` attribute of the file to match the publication date of the episode

If our implementation is written with testability (the concept) and specifically `Testability` (the package) in mind, we will have a very convenient way of giving our tests complete control over all of the external interactions our app has.

## The SUT

Let's start off with injecting our dependencies, defaulting them to the `Foundation` implementations:

{% highlight swift %}
struct PodcastArchiver {
    private let fetch: URLDataFetcher
    private let downloader: FileDownloading
    private let mover: FileMovement
    private let modifier: FileAttributes

    init(fetch: @escaping URLDataFetcher = URLSession.shared.data(from:),
         downloader: FileDownloading = URLSession.shared,
         mover: FileMovement = FileManager.default,
         modifier: FileAttributes = FileManager.default) {
        self.fetch = fetch
        self.downloader = downloader
        self.mover = mover
        self.modifier = modifier
    }

    func fetchAllPodcasts(from url: URL) async throws { ... }
}
{% endhighlight %}

By having `mover` and `modifier` as two separate things (even though in production they both point to the same `Filemanager` instance) our test implementations can be focussed on just one thing.

## The tests

With that in place, we can now write tests that:

 * can check the requests made are the correct ones
 * have full control over the data (or errors) delivered to the app
 * can check the movement of the download specifies the correct source and destination
 * can check that the attribute modification matches the date specified in the feed data

## Test implementations of dependencies

### URL logging

For "Mutation of captured var in concurrently-executing code" reasons, it's helpful to have a dedicated `actor` for the purposes of recording requested URLs:

{% highlight swift %}
actor RequestLogger {
    private(set) var urls: [URL] = []

    func log(_ url: URL) {
        urls.append(url)
    }
}
{% endhighlight %}

### Data fetching

Here we've used the minimal enclosure-based approach, so our implementation becomes very simple:

{% highlight swift %}
let responseData = """
    [
      {
        "episode": "https://some.podcast/some/episode.mp3",
        "title": "The very first episode",
        "published": "Mon, 10 Jan 2025 13:00:00 GMT"
      }
    ]
    """.data(using: .utf8)!

let logger = RequestLogger()

let fetch: URLDataFetcher = { url in
    await logger.log(url)
    return (responseData, HTTPURLResponse())
}

{% endhighlight %}

### File downloading

Using the same URL logger, we implement just the elements of `FileDownloading` that are required by the code under test.

{% highlight swift %}
class SpyDownloader: FileDownloading {
    private let logger: RequestLogger

    init(logger: RequestLogger) {
        self.logger = logger
    }

    func download(from url: URL, delegate: (any URLSessionTaskDelegate)?) async throws -> (URL, URLResponse) {
        await logger.log(url)
        return (URL(filePath: "/tmp/some/download.mp3"), HTTPURLResponse())
    }

    func download(for: URLRequest, delegate: (any URLSessionTaskDelegate)?) async throws -> (URL, URLResponse) {
        fatalError("Not exercised by SUT, so not implemented here")
    }
}
{% endhighlight %}

### File movement

Simply append to an array of tuples of source and destination for any requested move:

{% highlight swift %}
/// Keeps a log of *requested* moves. Never *actually* moves anything
class SpyFileMover: FileMovement {
    private(set) var moves: [(src: URL, dst: URL)] = []

    func moveItem(at srcURL: URL, to dstURL: URL) throws {
        moves.append((src: srcURL, dst: dstURL))
    }
}
{% endhighlight %}

### File attributes

Applying the rule that attributes of a file can only be read if any attribute was previously written to the same file, modify and return elements of an in-memory dictionary. This may not suit your needs - a more sophisticated solution would be to seed the Spy with a ready-made dictionary of files and attributes and consider all accesses to a "non-existant" file to be an error.

{% highlight swift %}
class SpyFileAttributes: FileAttributes {
    private(set) var attributes: [String: [FileAttributeKey : Any]] = [:]

    struct FileNotFoundError: Error {}

    func setAttributes(_ attributes: [FileAttributeKey : Any], ofItemAtPath path: String) throws {
        for (key, value) in attributes {
            self.attributes[path, default: [:]][key] = value
        }
    }

    func attributesOfItem(atPath path: String) throws -> [FileAttributeKey : Any] {
        guard let attributes = attributes[path] else {
            throw FileNotFoundError()
        }
        return attributes
    }
}
{% endhighlight %}

### The test(s)

In reality, this wouldn't be written as one giant test, but here it serves to illustrate the point that the tests:

 * are in complete control over, and
 * have total visibility of all of our code's interactions

{% highlight swift %}
@Test func comprehensiveExample() async throws {
    // GIVEN
    let responseData = """
    [
      {
        "url": "https://some.podcast/some/episode.mp3",
        "title": "The very first episode ever!",
        "published": "Mon, 10 Jan 2025 13:00:00 GMT"
      }
    ]
    """.data(using: .utf8)!

    let logger = RequestLogger()
    let spyDownloader = SpyDownloader(logger: logger)
    let spyMover = SpyMover()
    let spyAttributes = SpyFileAttributes()

    let sut = PodcastArchiver(
        fetch: { url in
            await logger.log(url)
            return (responseData, HTTPURLResponse())
        },
        downloader: spyDownloader,
        mover: spyMover,
        modifier: spyAttributes
    )

    // WHEN
    try await sut.fetchAllPodcasts(from: URL(string: "https://some.podcast/feed.json")!)

    // THEN
    // There should be exactly 2 requests
    #expect((await logger.urls.count) == 2)
    #expect((await logger.urls.first?.absoluteString) == "https://some.podcast/feed.json")
    #expect((await logger.urls.last?.absoluteString) == "https://some.podcast/some/episode.mp3")

    // There should be exactly 1 move
    #expect(spyMover.moves.count == 1)
    #expect(spyMover.moves.first?.src.path(percentEncoded: false) == "/tmp/some/download.mp3")
    let expectedDstPath = "/Users/chris/Documents/The very first episode ever!.mp3"
    #expect(spyMover.moves.first?.dst.path(percentEncoded: false) == expectedDstPath)

    // The created date should have been set
    #expect(spyAttributes.attributes[expectedDstPath]?[FileAttributeKey.creationDate] as? Date == Date(timeIntervalSince1970: 1736514000))
}
{% endhighlight %}

Of course we now need an implementation to test. For that, see [the example in the package][2].

[1]: https://github.com/ChrisLawther/Testability
[2]: https://github.com/ChrisLawther/Testability/blob/main/Tests/TestabilityTests/Comprehensive/PodcastArchiver.swift

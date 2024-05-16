---
author: Kevin Kelly
title: "Boomic Architecture: Composers and Instruments"
date: 2024-05-02
description: The general architecture of Boomic Music.
tags:
  - software
  - software-design
  - boomic
  - swift
categories: 
series: 
aliases: 
draft: false
---
The first time I set out to make Boomic, I didn't think about architecture at all. The app quickly became a coupled disaster, and making the smallest changes felt daunting. This time around, I committed to learning and understanding software design and architecture. To learn to build a composable system, designed for change. This is how I started, where I went wrong, and where I am now.

I ended up starting my development based on the layered architecture from Jacob Bartlett's excellent post on [Modular Architecture for Apps](https://jacobbartlett.substack.com/p/modular-architecture-for-apps). I was closely following the architecture of their [Bev Sample App](https://github.com/jacobsapps/bev17/tree/main?tab=readme-ov-file), using the Repository pattern, featured in his series on [Async Unit Testing](https://jacobbartlett.substack.com/p/async-unit-testing-in-swift-the-comprehensive). I highly recommend [Jacobs Tech Tavern](https://jacobbartlett.substack.com/) to any iOS app developers.

The 'Repository Pattern' is where you create a single `Repository` component that serves the data to the UI. The UI doesn't have to worry about where the data is coming from, while the repository can combine data from different sources. In Bev, Jacob shows a use case of grabbing local, cached data from a database, while fetching the most up-to-date data from an API. Here's cut down version of what this looks like in Bev:

```swift
private func returnMultipleTimes() async throws {
	
	if let beers = try? await db?.getBeers() {
		beersPublisher.send(beers)
	}
	
	do {
		let beers = try await api.getAllBeers()
		try? await db?.save(beers: beers)
		beersPublisher.send(beers)
	} catch { /* handle error */ }
}
```

My first goal for Boomic was to simply load audio files from a local directory into a `Database` as `Song` objects, then load them from the database onto the UI. I needed some kind of `MediaFileInterface` to load audio files from a root directory and extract their metadata into the `Song` objects. Thinking that the database was likely the only thing that needed to load files this way, as the rest of my app could just use the songs from the database, I went with a design like this:

![Original Database with coupled file interface](/blog/images/original_database.svg)

It was after watching Rich Hickey's [Design, Composition, and Performance](/blog/learning/design_composition_performance.md) that I realized what kind of mistake I was making with the database. 

I thought that simply by using Swift's protocols, I was keeping these components decoupled. I could easily switch from a local to a remote directory with the same `MediaFileInterface` protocol. But I was still coupling the `Database` to the fact that it had to know where it was getting its `Song` objects from. This had flown under my radar until I thought about Apple Music. A long-term goal of Boomic is to be able to sync your Apple Music library alongside your local library. Since I want to have the same tags and ratings for both Apple Music and local songs, I wanted to store them in the same database. Assuming I did that that now, the architecture would look like this:

![Original Database with two coupled file interfaces](/blog/images/original_database_expanded.svg)

I would need to edit the `Database` itself, its public interface, and the repository that calls it. By just making the database aware of a protocol to get new songs, I was giving the database an extra responsibility: don't just store the songs, but get the new ones as well. Lets remove that separate responsibility and see what it looks like:

![Update, decoupled database with separate file interface](/blog/images/decoupled_database.svg)

Now, the `Repository` can simply have two different functions for each different source. The `Database` itself can be designed in a much simpler, more generic manner, without worrying where it is getting new data from. More importantly, when I want to add the Apple Music support, I now only have to change the app-specific `Repository`. I simply a create a new composition of a new component, the `AppleMusicInterface`, and an old one, the `Database`.

---
### Breaking it down: Composers and Instruments
I really liked the general repository pattern, and felt it could be extended outside the realm of data operations. As I mentioned earlier, it was watching [Design, Composition, and Performance](/blog/learning/design_composition_performance.md) when the value of the repository pattern really clicked for me. So, lets borrow some vocabulary (albeit with a slightly different use) and see how it works:

- **Instruments:** Single responsibility, usually generic components.
- **Composers**: Components that coordinate one or more instruments to achieve something specific.

In my app so far, the `Database`, `MediaFileInterface`, `AppleMusicInterface`, and any other sources of data such as an `ImageCache` are the instruments. Each instrument does their specific tasks, is completely decoupled from other instruments, and can be reused in other parts of the program. Then, the `Repository` is a composer that handles the specific needs of my app by, well, composing these instruments together.

---
### Composers as Instruments.
I want to make one thing about this design philosophy clear: Just because something is an instrument, doesn't mean it cannot be composed of other instruments. An example of this is how I am currently storing transactions.

To store transactions as they're received, I built a generic `LogStore` class that saves `Codable` objects at a key in the order they're received, like a standard log. This is pretty clearly an instrument: single-responsibility, generic, ready to be composed into various purpose specific components. But the `LogStore` itself was composed with another instrument: the `SimpleStore`.

The `SimpleStore` itself just saves and overwrites the object saved at its key. So the most basic implementation of a `LogStore<T>` can just use a single `SimpleStore<[T]>`, which is exactly what I have done for now as my proof-of-concept:

``` swift
public final class LogStore<Log: Loggable> {
    
    private let storage: SimpleStore<[Log]>
    
    public init(key: String) {
        self.storage = SimpleStore(key: key)
    }
    
    public func save(_ log: Log) async throws {
        let pastLogs = try await storage.load()
        var logs = pastLogs ?? []
        logs.insert(log, at: 0)
        try await storage.save(logs)
    }
    
	/* ... */
}
```

This is pretty inefficient, since as logs are written to disk right away. A better version of this would be using multiple simple stores, with a cache `SimpleStore` for quickly saving of the most recent logs, which are periodically merged them in with the older logs from another `SimpleStore` in the background. Either way, the `LogStore` here an example of a composer: composing one or more instances of the generic `SimpleStore` instrument. But, it itself is a generic, composable instrument, that I use to store transactions now, and probably logs in the future.

I think Rich Hickey really illustrates this point well at from [41:19-42:42](https://youtu.be/QCwqnjxqfmY?t=2479&si=jhGajdysPBKZE5As) in, you guessed it, Design, Composition, and Performance! [Yves Usson](https://www.arturia.com/stories/yvesusson-insideamodularmind), who designed the synthesizer, composed the synthesizer it out of machines that themselves are individual instruments, or at least would be in this current mental model. Then, the synthesizer itself is an instrument available to musicians to be composed with other instruments.

---
### Isn't this obvious?
Isn't this just another way of saying 'we should be building composable systems?' Yeah, pretty much, but I think there is a lot of value in reframing the way we think about our systems with specific terminology and strong analogies. In just writing this post and applying this thinking to the system that inspired it, I realized that the `MediaFileInterface`, which doesn't necessarily violate the Single Responsibility Principle (translate all the files in a root directory into `Song` objects), is both not generalizable, and could be broken apart into two separate, more generalizable, instruments. 

``` swift
// old
let newSongs = MediaFileInterface().newSongs(existing: existingSongs)

// new
let knownFiles = existingSongs.map { $0.file }
let newFiles = FileInterface().all(["mp3", "flac"], ignoring: knownFiles)
let newSongs = newFiles.map { Song(fromFile: $0) }
```

For me, it really is that reframing that can bring a lot of value. I was comfortable calling the `MediaFileInterface` a simple, single responsibility component that only had one public function. And it was only 60 lines. But it was a specific component I built for a specific purpose. When I want to play guitar in a new key, do I buy a whole new guitar? No, I take my existing guitar, and just put on, *or compose it with,* a capo!

So, after reframing the question, I was not comfortable calling the `MediaFileInterface` a good instrument. Which led me to asking "Could this be composed of simpler instruments?" and "Could this be more generic?" and ended with me building the new, more generic `FileInterface`, which I am already using in another part of the `Repository`.

To summarize, I think we can build the vast majority of what we create with generic instruments and a few composers. Whenever we do something specific, we should try to break it down into generic instruments, and build those instruments up with simpler and simpler instruments. This will give us many a huge toolbox of generic, testable instruments, which can easily be rearranged to make changes.

---
### In Practice
Finally back to the app, what does Boomic look like today. I am still some time away from the MVP, and so plan on updating this part of the post down the road. Especially if (more likely when) I find I didn't have as complete of an understanding as I thought I had. But here is the current structure. Relatively flat, but much, much less coupled then its predecessor:

![boomic_architecture](/blog/images/boomic_arc.svg)

- **`Repository`**: Composer of the data instruments, specifically for the UI.
	- **`Transactor`**: Serially stores library transactions, and publishes a new `DataBasis` to reflect that transaction.
		- **`DataBasis`**: Immutable value of the entire library.  Progression of state tracked by the `Transactor`.
	- **`QueryEngine`**: Queries a `DataBasis` value.
	- **`FileInterface`**: Provides enumerations of specified file extensions from a root directory.
	- **`MetadataLoader`**: Extracts metadata from audio files, both embedded and from the surrounding files.
- **`Player`**: Composer of the audio playback instruments. Has a fair amount of state.
	- **`Engine`**: The interface to the library that makes the system calls to playback audio. Each song gets its own engine, allowing for pre-loading. Has a fair amount of state.
	- **`Queue`**: An immutable value representing the queue. Progression of state tracked by the `Player`.
	- **`Mixer`**: Applies EQ and Effects to the `Engine`.
- **`ImageCache`**: Shared cache, keys embedded album art with MD5 hashes to prevent initializing the same images over and over.
##### Immediate Plans
- Create a `LiveQueryEngine` that tracks changes and updates views when the `DataBasis` updates.
- Have the `Player` directly access the `Repository` instead of just sharing the `ImageCache` so it can respond to `DataBasis` updates.
- Look into making the `Engine`, `Queue`, and `Transactor`, classes more generic. They all currently rely on this app's custom data types, but don't necessarily need to.

There is a lot more work to do to make the immutable `DataBasis` pattern work with Swifts observation frameworks, and make the UI reactive to specific changes. I have reflected on whether keeping the pattern is worth it and have decided to continue it. More on that in a future post.

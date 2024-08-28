---
author: Kevin Kelly
title: "Programming a Symphony: Instruments, Performers, and Composers."
date: 2024-08-26
description: Think
tags:
  - software
  - software-design
  - software-quality
  - software-complexity
categories: 
series: 
aliases: 
draft: false
---
Writing software is easy. Writing good software for large projects is anything but. 

What 'good' software then? Some of the characteristics that come to mind are simple and easy to understand, adaptable and easy to maintain, and testable. But knowing these characteristics doesn't tell us anything about *how* to write good software.

I certainly don't have all the answers, and if I did, I'd be working on my suntan on my private beach as opposed to writing this post. But what I can share with you is one of the big breakthroughs in my personal work, which I call the *orchestra heuristic*. I picked up the idea from Rich Hickey's [*Design, Composition, and Performance*](https://www.youtube.com/watch?v=QCwqnjxqfmY) Keynote. This 51 minute presentation does a great job talking about the craft of software at a high level, covering many more ideas then I will here, and I highly recommend you watch it in full (really, I recommend you watch all his keynotes, twice). But I wanted to take some time to go over some concrete code examples in my preferred language of Swift to demonstrate how powerful this single idea has been for me.

# The 'Hardware' Orchestra
Before we can dive into how thinking about a an orchestra can help us write better software, we need to break down the levels of a actual orchestra. Afterwards, we can think about the ways in which it's similar to our software problem.

#### The Instruments
You can't have music without instruments (well, you can, but odds are it won't sound as good). Instruments are designed to play a range of sound in different contexts. Some are highly focused and have small range, some are highly dynamic and have a wide range. Sometimes they can even be tuned to different keys/tones. Through an *interface*, they produce their desired result, the vibrations in the air we perceive as sound.

#### The Performers
Those instruments may look beautiful just sitting there, but they need dedicated performers to create the sound. The performers might not know anything about the physics behind the sound of their instruments, or know the first thing about how to play the instruments of the performers sitting next to them. Even if they do, they don't think about when they're focused on their performance. During their performance, they're intimate experts of the interface of their own instruments, wholly focused expressing themselves their role in the symphony.

#### The Conductor
Finally, we have a conductor at the top of the orchestral *stack*. The one who ties it all together. A group of performers, no matter their talent, will struggle to coordinate their sound while they're so wholly focused on making their own contributions perfect.  

When we do have a conductor up there, they're not thinking about the individual performances or instruments being used. They're entirely focused on the symphony as a whole: rhythm, harmony, tempo, etc. The entirety of the performance is represented through the *interface* of sheet music.

#### The Composer
Did you forget about the composer? Someone had to write the music. Though often different people, in terms of this analogy, we are going to assume the conductor also composed the music. And as it works better for the analogy, I will refer to both as the 'Composer' from now on.

# The 'Software' Orchestra
With that out of the way, lets note some of the software parallels already found in our Hardware Orchestra example:

- Interfaces: The Performers use their instruments through the instrument's interface, without any need to understand the underlying physics that make the sound. And the composer 'uses' their performances through the interface of the sheet music, without needing to understand the details, such as finger placement or technique, of any performance. It can certainly help to know these things, but it's best to leave them out of mind when it comes time to lock in and perform your part.
- Stack: Not the data structure, more of a call stack. The composer at the top level, 'calls' the performers to play the written music, while the performers themselves 'call' their instruments to create sound through the interface.

So what might the components of our 'Software Symphony' look like? 

#### A Software Instrument
Like our hardware instruments, software instruments are usually generic, capable of achieving a range of tasks through their interface, and are useful in a range of contexts. It's this range and *context independence* that naturally promotes code reuse. 

This doesn't necessarily mean 'for use in any system.' It could be as simple as 'for use in multiple parts of this system.'

Some examples of software instruments include:
- Data Structures, such as arrays, sets, and dictionaries.
- Generic UI components. In `SwiftUI`, this might be `VStack`, `Text`, `Button`, etc. Or app specific `SongGrid`, `LargeButton`, etc.
- External Interfaces, such as a `FileManager`, an API, or a Database connection. 

#### A Software Performer
The software performer is what we too often first jump to when trying to solve a problem: a function/class, purpose built to solve the problem at hand. That is the key difference between an instrument and a performer: the performer is designed for a specific context. 

If we wanted to use a binary tree to solve a problem, we could use an array to represent the binary tree, and write private 'performer' functions that maintain the tree with whatever type we are using in that context. Or, we instead could create a 'instrument' binary tree class and directly use it in our problem. With the latter, not only will the solution of the original problem become easier to use, understand, and test, but now we have a generic instrument we can reuse in different contexts. 

Some examples of software performers include:
- A function to call a specific API to fetch data for the logged in user.
- A UI view that draws out a login screen for an app.
- A class that represents a user for an app.

#### A Software Composer
A software composer is anything uses multiple instruments or performers to do a task. The key is recognizing that there are multiple instruments/performers in use, and then to treat the composer with the respect it deserves by factoring each instrument/performer out. 

What we will then be left with are short blocks, or even just a few lines of code that describe how the code works in comment like clarity. The performers we create will make our current context more composable, and the instruments we create will help our entire codebase.

Some examples of software performers include:
- A Repository class that combines data from difference sources, such as a local copy and new data from an API.
- A function that compares two result sets, but calling other functions to filter each result set.

# The Differences
Before we dive into some examples, we should clear up some of the ways this analogy doesn't hold up as a 1-to-1 comparison. For example, in our hardware orchestra, each performer only plays one instrument. We know that's not how it works in software land: A single software performer may use multiple instruments, or even other performers, to achieve its goals. And most projects are more than 3 layers deep, and coordinate at multiple layers as opposed just having monolithic composer at the top of the stack.

Another difference is that this analogy is very context dependent: what you write in a composer mindset might be a performer or an instrument in another context. For example, the Repository pattern, which we will discuss in an example, composes different instruments/performers into a single interface for data. So we want to write it in a composer mindset, delegating most processing to its instruments/performers. But when we use it in the UI, it turns into a instrument, with the responsibility of getting our app data.

The real goal of this heuristic is to align your mindset with the context you're currently working in. If you're writing a complex class or function that composes many components together, you probably want to be writing that code with a composer mindset. When you're writing a performer that does something specific, you should be thinking if it is doing more then one thing and become a composer, or could be more generic and become an instrument. And if you're writing an instrument, you should be totally ignoring the context you began with, boiling down the solution into its core, generic components.

Overall, the lines between instruments, performers, and composers in software are pretty blurry. Partly because everything we build is built with code, it can be hard to recognize the different needs of each context.  So let's work through some examples to see if we can get the hang of it.

# Example 1: Finding the k-th Max Element.
The first example I want to show is a LeetCode problem, specifically [#215. Kth Largest Element in an Array](https://LeetCode.com/problems/kth-largest-element-in-an-array/). One of the major learning blindspots from only doing LeetCode problems is the focus on isolated problems: each LeetCode solution is a performer. But we can still practice creating the context independent instruments and maintainable composers that will make the larger projects we work on much easier to manage.

Lets start with an 'unrolled' version of a solution, a pure performer. Here, we are using a [MinHeap](https://www.geeksforgeeks.org/introduction-to-min-heap-data-structure/) to solve the problem, but we are storing in an array called `heap`.

``` swift
class Solution {
    func findKthLargest(_ nums: [Int], _ k: Int) -> Int {
        var heap: [Int] = []

		// Insert the number into the heap.
		for number in nums {
			heap.append(number)

	        var numberIndex = heap.lastIndex
	        var parentIndex = parent(of: numberIndex)
	
	        while (heap[numberIndex] < heap[parentIndex]) {
	            swap(numberIndex, parentIndex)
	
	            numberIndex = parentIndex
	            parentIndex = parent(of: numberIndex)
	        }
		}

		// Remove the root (minimum number)
		for number in 0..<k {
			swap(0, storage.lastIndex)
	        storage.remove(at: storage.lastIndex)
	
	        var parentIndex = 0
	        while let swapIndex = parentSwapIndex(parentIndex) {
	            swap(parentIndex, swapIndex)
	            parentIndex = swapIndex
	        }
		}
    }

	// Find the parent index of an index in the heap
	private func parent(of index: Int) -> Int {
		return (index - 1) / 2
	}

	// Swap the values of two Indices
	private func swap(_ a: Int, _ b: Int) {
        let aValue = storage[a]
        storage[a] = storage[b]
        storage[b] = aValue
    }

	// If the parent is greater then one of its children, it
	// needs to be swaped. This returns the index to swap with
	// or nil if it doesn't need to be swaped.
	private func parentSwapIndex(_ parentIndex: Int) -> Int? {
        // left blank for brevity
    }
}
```

Now this isn't too bad, but lets look at is through our orchestral lense. The `findKthLargest()` function is a performer: a specific solution to a specific problem. We have three helper functions defined as well, all of which I'd define as performers. They're fine though: they each have one simple, understandable task. While our `findKthLargest()` does what it says, it is doing a lot. Lets try out the composer mindset, and see if we can compose something out of a few performers, giving comment like clarity.

``` swift
class Solution {
	func findKthLargest(_ nums: [Int], _ k: Int) -> Int {
        var heap: [Int] = []

		for number in nums {
			heap = insert(number, into: heap)
		}

		for number in 0..<k {
			heap = removeRoot(from: heap)
		}

		return heap[0]
	}
}
```

Thats a lot better. When we are writing with the composer mindset, we should be thinking of making the steps as clear as possible: doing all of the work using simple performers. This code basically reads like the comments, so we can remove them.

But lets not forget about other end of the orchestral stack, and reexamine what we have. What instruments have we using? Well, while we logically know the data structure is a MinHeap, what we're really doing is using a simple `Array` and doing a *lot* of performing to turn it into a MinHeap. Like trying to play a Beethoven masterpiece on a 24 key plastic children's keyboard. It's impressive, and probably would make for great 'content.' But we're serious engineers trying to get things done. We need to shift into the instrument mindset, and build ourselves some serious tools.

``` swift
class Solution {
	func findKthLargest(_ nums: [Int], _ k: Int) -> Int {
        var heap: MinHeap<Int>()

		for number in nums {
			heap.insert(number)
		}

		 while heap.count<k {
			heap.removeRoot()
		}

		return heap.root
	}
}

// Public interface only
class MinHeap<Element: Comparable> {
	var root: Element?
    var count: Int 
    func insert(_ element: Element) 
    func removeRoot()
}
```

This is the solution that comes out of searching for the best instruments that describe our problem. No need to compose two specialized, context specific performers. We now have a generic `MinHeap` instrument we can use whenever we need that data structure on anything that conforms to `Comparable`. 

Is our final result even a composer anymore? This is another place where the analogy gets fuzzy. You could argue it's now a performer, with it simply interfacing with a single instrument. But I'd still consider it a composer, as it still follows the spirit of a composer: laying out what it does in comment like clarity.

# Example 2: Extracting Metadata from an Audio File
Lets move into to a real world example with the logic I am currently using to read audio file metadata tags in [Boomic Music](https://github.com/kevink2022/Boomic), an app I'm currently working on. I will warn you, I should be using a third party library (like [TagLib](https://taglib.org/)) for this, and not Apple's ancient `AudioToolbox` library. But I do think this is a great example how coding with this mindset naturally leads into easy refactoring and adaptability, so it does make a pretty good example. 

The following code is really long - really meant to demonstrate how code can balloon if we aren't careful. Feel free to just skim the comments to get an idea of what it is doing.

``` Swift
private static func song(from file: URL) -> Song {
	
	// 1. Extract the internal tags from the file
	var fileID: AudioFileID? = nil
	var tags: [String : Any] = [:]
	var embeddedArtHash: String? = nil
	
	let fileOpenStatus: OSStatus = AudioFileOpenURL(
		file as CFURL
		, .readPermission
		, kAudioFileMP3Type
		, &fileID
	)
	
	if fileOpenStatus == noErr, let fileID = fileID {
		defer { AudioFileClose(fileID) }
		
		// 1a. Extract Embedded Tags
		var dict: CFDictionary? = nil
		var dataSize = UInt32(
			MemoryLayout<CFDictionary?>.size(ofValue: dict)
		)
		
		let getPropertyStatus: OSStatus = AudioFileGetProperty(
			fileID
			, kAudioFilePropertyInfoDictionary
			, &dataSize, &dict
		)
		
		if getPropertyStatus == noErr, let cfDict = dict {
			let tagsDict = NSDictionary.init(dictionary: cfDict)
			tags = tagsDict as? [String : Any] ?? [:]
		}
		
		// 1b. Extract Embedded Album Art
		let embeddedArtStatus = AudioFileGetPropertyInfo(
			fileID
			, kAudioFilePropertyAlbumArtwork
			, &dataSize
			, nil
		)
		
		if embeddedArtStatus == noErr {
			var artworkData: UnsafeMutableRawPointer? = nil
			
			let result = AudioFileGetProperty(
				fileID
				, kAudioFilePropertyAlbumArtwork
				, &dataSize
				, &artworkData
			)
			
			if result == noErr, let artworkDataUnwrapped = artworkData {
				let cfData = Unmanaged<CFData>
					.fromOpaque(artworkDataUnwrapped)
					.takeRetainedValue()
					
				let data = Data(referencing: cfData)
				embeddedArtHash = HashUtility.computeHash(data: data)
			}
		}
	}
	
	// 2. Check for Album Art in Album Directory
	let fileManager = FileManager()
	// This is an assumption we're making for this example
	let directoryURL = file.deletingLastPathComponent() 
	var externalArt: URL?
	
	do {
		let filesInDirectory = try fileManager.contentsOfDirectory(
			at: directoryURL
			, includingPropertiesForKeys: nil
			, options: []
		)
		
		let artworkFiles = filesInDirectory.filter { url in
			let fileExtension = url.pathExtension.lowercased()
			let fileName = url
				.deletingPathExtension()
				.lastPathComponent
				.lowercased()

			let isCover = ["cover", "folder", "album"].contains(fileName) 
				&& ["jpg", "png"].contains(fileExtension)
			
			if isCover {
				return true
			} else {
				return false
			}
		}
		
		externalArt = artworkFiles.first
		
	} catch { /* No errors in example land */}
	
	// 3a. Determine album art
	var albumArt: MediaArt? = nil
	
	if let embeddedArtHash = embeddedArtHash {
		albumArt = .embedded(
			path: AppPath(url: file)
			, hash: embeddedArtHash
		)
	} else if let externalArt = externalArt {
		albumArt = .local(path: AppPath(url: externalArt))
	}
	
	// 3b. Init and return song
	let fileName = file.lastPathComponent
	let durationTag = "approximate duration in seconds"
	
	return Song(
		id: UUID()
		, source: .local(path: AppPath(url: file))
		, duration: TimeInterval(tags[durationTag] as? String ?? "") ?? 0
		, title: tags["title"] as? String ?? fileName
		, trackNumber: Int(tags["track number"] as? String ?? "")
		, discNumber: Int(fileName.split(separator: "-")[0])
		, art: albumArt
		, artistName: tags["artist"] as? String
		, albumTitle: tags["album"] as? String
	)
}
```

So what is it doing?

First, we get the metadata the embedded in the file, including the tags and the album art. The instrument we are performing with here is the `AudioToolbox` library. It may look like a bunch of performers, being separate functions and not a single class. But they are context independent, and would look at more familiar if we rewrote the functions to be called on an `AudioFile` class instead of being called on passed `fileID`.

Second, we check if theres any album art outside the file itself, specifically in the same directory. Often with large albums, instead embedding the same image in each song file, a 'cover' image will be included in the album directory to save space. We also check the filename for a disc number. Our instrument here is the `FileManager`. 

Finally, we initialize and return the user defined `Song` structure.

Right away, aside from the busy nature of the function (outside of the UI, there is not a single function in the entire 10k+ line project longer then this one), I notice the composition of a few distinct performances: one getting embedded data with `AudioToolbox`, another getting external data with `FileManager`, and finally one combining that data together to create a `Song`. So the goal is now to refactor it in a way that simply describes the performances it coordinates. So we can try factor out what we can:

``` swift
static func songBetter(from file: URL) -> Song {
        
	// 1. Extract the internal tags from the file
	var fileID: AudioFileID? = nil
	var tags: [String : Any] = [:]
	var embeddedArtHash: String? = nil
	
	let fileOpenStatus: OSStatus = AudioFileOpenURL(
		file as CFURL
		, .readPermission
		, kAudioFileMP3Type
		, &fileID
	)
	
	if fileOpenStatus == noErr, let fileID = fileID {
		defer { AudioFileClose(fileID) }
		
		tags = embeddedTags(from: fileID) ?? [:]
		embeddedArtHash = artHash(from: fileID)
	}

	// 2. Check for Album Art in Album Directory
	let externalArt = try? externalArtFile(for: file)

	// 3. Determine Album Art
	let albumArt = albumArtFrom(
		for: file
		, embeddedHash: embeddedArtHash
		, externalFile: externalArt
	)
	
	// Init and return song
	let fileName = file.lastPathComponent
	let durationTag = "approximate duration in seconds"
	
	return Song(
		id: UUID()
		, source: .local(path: AppPath(url: file))
		, duration: TimeInterval(tags[durationTag] as? String ?? "") ?? 0
		, title: tags["title"] as? String ?? fileName
		, trackNumber: Int(tags["track number"] as? String ?? "")
		, discNumber: Int(fileName.split(separator: "-")[0])
		, art: albumArt
		, artistName: tags["artist"] as? String
		, albumTitle: tags["album"] as? String
	)
}
```

We've made a lot of progress here, this can now generally be read and understood. Down from 80+ lines to just over 30. But once again, we have made context specific performers where we could have had context independent instruments. 

Lets think about the three performers we identified earlier. 

First, we use `AudioToolbox` to get embedded metadata. Why not a generic instrument for retrieving embedded tags? 

Second, we currently have our rules for the external tags all over the place, with the album art rules in our `albumArtFor()` function, and the disc number being extracted from the file name in the `Song` init itself. That can be another instrument, with all the rules on what defines an 'external tag' defined in one place.

Finally, we initialize the song with its default initializer. Well, if the purpose of this performer is it initialize a new song, why aren't we making this an initializer itself?

``` swift
public convenience init(from file: URL) {
	let internalReader = AudioToolboxMetadataReader(file: file)
	let externalReader = DirectoryMetadataReader(file: file)
	
	let albumArt: MediaArt? = {
		if let hash = internalReader.embeddedHash { 
			return .embedded(path: AppPath(url: file), hash: hash) 
		} if let art = externalReader.albumArt { 
			return .local(path: AppPath(url: art)) 
		}
		return nil
	}()
	
	self.init(
		id: UUID()
		, source: .local(path: AppPath(url: file))
		, duration: internalReader.duration ?? 0
		, title: internalReader.title
		, trackNumber: internalReader.trackNumber
		, discNumber: externalReader.discNumber
		, art: albumArt
		, artistName: internalReader.artist
		, albumTitle: internalReader.album
	)
}
```

Can't get much simpler then that. We now have two context independent instruments for reading metadata tags from an audio file, and it's clear where the data for each field is coming from when we initialize the `Song`.  

While unrolling this final block of code to create the first one, I even left out some of the issues, it really should be longer. For example, there's an fix for extracting the track numbers I left out, a large portion of my own library would fail to initialize their track numbers here. In each of the three steps we took to write this function, where would you look to find it? What tests would you write to test it? In the first one step, it would be a pain, while in the final step, its obvious where to look. 

And when I finally grow up and find/create a TagLib C++ to Swift bridge to use, it will be as simple as switching from the `AudioToolboxMetadataReader` to the `TagLibMetadataReader`. Have fun doing that if your code looks like the first step! 

Additionally, by removing the context of the current problem, we can write much cleaner code *within* each instrument. For the `AudioToolboxMetadataReader`, if the file doesn't open, we have nothing to do. In the original example, we need to keep the full problem in mind, so we can't return early and we have to nest everything in a huge `if let` block. But our instrument doesn't care about the larger problem, so we can switch do a `guard let` and simply return early.

``` swift
init(file: URL) {
	guard let fileID = Self.openFile(url: file) else {
		tags = nil
		embeddedHash = nil
		return
	}
	
	defer { Self.closeFile(fileID) }
	tags = Self.initTags(from: fileID)
	embeddedHash = Self.embeddedArtHash(from: fileID)
}
```

And when we're initializing the tags, we can use the same `guard let` pattern to achieve zero nesting in our happy path where nothing fails.

``` swift
private static func initTags(_ fileID: AudioFileID) -> [String : Any]? {
	var dict: CFDictionary? = nil
	var dataSize = UInt32(MemoryLayout<CFDictionary?>.size(ofValue: dict))
	
	let getPropertyStatus: OSStatus = AudioFileGetProperty(
		fileID
		, kAudioFilePropertyInfoDictionary
		, &dataSize
		, &dict
	)
	guard getPropertyStatus == noErr else { return nil }
	
	guard let cfDict = dict else { return nil }
	let tagsDict = NSDictionary.init(dictionary: cfDict)
	return tagsDict as? [String : Any]
}
```

Now, if you while reading the third step you had the thought "Why not make a single `MetadataReader` that composes the two sources together?" you're starting to get it. It's hard to imagine many scenarios where I would need to use just one or the other. But if I did, I would still have that option. Life is easy when you have an orchestra of instruments at your disposal.

# Example 3: Enumerating Files
Another example from Boomic, but don't worry, the rest the examples are much shorter. I had built an instrument for enumerating files, called the `FileInterface`. A few uses included enumerating the whole library of audio files, enumerating an album's directory, and enumerating a folder of scans included with an album, like the back of a vinyl sleeve and any inserts.

Here's an example of getting new files to import into the library:

``` swift
let existingAudioFiles = self.songs.map { $0.fileURL }

let newAudioFiles = FileInterface(at: Song.codecs)
	.enumerate(at: URL.documents, excluding: Set(existingAudioFiles))
```

Originally, I had built the `FileInterface` to use `URLs`:

``` swift
func enumerate(of extensions: Set<String>? = nil, excluding: Set<URL>? = nil) throws -> [URL] {
	guard let enumerator = fileManager.enumerator(
		at: root
		, includingPropertiesForKeys: nil
		, options: .skipsHiddenFiles
	) else { 
		throw FileInterfaceError.enumeratorInitFail(root)
	}
	
	let allURLs = enumerator.allObjects.compactMap { $0 as? URL }
	let allExtensionURLs = allURLs.filter { url in
		extensions?.contains(url.pathExtension.lowercased()) ?? true
	}
	
	let allNewURLs = {
		if let excluding = excluding, excluding.count > 1 {
			return allExtensionURLs.filter{ !excluding.contains($0) }
		} else {
			return allExtensionURLs
		}
	}()
	
	return allNewURLs
}
```

This is a little busy for what's really a composer, but the clear constant names make up for that a lot. But we need to add a new `enumerate()` function. Basically, all iOS apps run in an isolated container, and when that container changes, any file URLs in the database will break. So I created a `AppPath` instrument that would represent a 'container agnostic' URL, so we need to add a new `enumerate()` that supports it.

Now that we'd be be duplicating a lot of that code, we can finally give these composers the respect they deserve.

``` swift
func enumerate(of extensions: Set<String>? = nil, excluding: Set<URL>? = nil) throws -> [URL] {
	let allURLs = try allFiles()
	
	let allExtensionURLs = filterExtensions(
		include: extensions
		, in: allURLs
	)
	
	let allNewURLs = filterExcludedURLs(
		exclude: excluding
		, from: allExtensionURLs
	)
	
	return allNewURLs
}

func enumerate(of extensions: Set<String>? = nil, excluding: Set<AppPath>? = nil) throws -> [URL] {
	let allURLs = try allFiles()
	
	let allExtensionURLs = filterExtensions(
		include: extensions
		, in: allURLs
	)
	
	let allNewURLs = filterExcludedAppPaths(
		exclude: excluding
		, from: allExtensionURLs
	)
	
	return allNewURLs
}
```

I like how this example demonstrates how composition becomes easy when we really break down our composers into their performers. While it was the code repetition that triggered the necessary decomposition of the original, our goal is not to simply minimize the amount of code written: 3/4 lines of each function are the same. Instead, we focused on creating single responsibility performances, which we then composed in two different ways to achieve the desired functionality.

# Example 4: The Repository Pattern
Last but not least, let look at an entire class built with these composer principles. I have to give credit to [Jacob Bartlett](https://jacobbartlett.substack.com/) in his [Async Unit Testing Guide](https://jacobbartlett.substack.com/p/async-unit-testing-in-swift-the-comprehensive) series for not just teaching me this pattern, but basically getting me started with iOS app development.

Basically, the Repository Pattern is where you create a class to be single interface/point of contact with your UI/frontend. The UI doesn't have to worry about where the data is coming from, whether its from an online API, locally stored in a database, sourced from local files, or even a combination of all three!

By nature, this class is a composer, coordinating the various sources of data into a single interface. For example, lets look at function Jacob Bartlett's `Repository` class from his sample [Bev](https://github.com/jacobsapps/bev17) app from the aforementioned unit testing guide.

``` swift
private func upToDateWithFallback() async throws {
	do {
		let beers = try await api.getAllBeers()
		try? await db?.save(beers: beers)
		beersPublisher.send(.success(beers))
		
	} catch {
		if let beers = try? await db?.getBeers() {
			beersPublisher.send(.success(beers))
		} else {
			throw error
		}
	}
}
```

This is a `DataAccessStrategy` for the function `getBeers()` in Bev's Repository. Highly recommend checking out the source code for the full example of the data access strategy pattern. In this strategy, we try and grab the most recent data from the API, but if that fails, we check the database for a saved backup. Every time the API is returns new data, including in the other strategies, we save that data in the database.

"Every time" you say? Smells like ugly code repetition. In fact, there's not a single strategy that doesn't involve both the API and the database. Wouldn't be nice if we made a single `CachedAPI` class to encapsulate that functionality?

That kind of thinking optimizes for *now.* But we aren't sure how the requirements might change in the future. Will we have to use another API? Move our database to the cloud, and therefore need stronger error handling? What if our database expands to include user data? What if we start importing from local files? 

By keeping each instrument/performer completely separate, and just composing them in the `Repository`, we limit the scope of each of the aforementioned potential changes to the single class and the `Repository` itself. And the `Repository` changes would likely be nothing more then some short composition functions. 

Like a good composer, the `Repository` code pretty much explains what it doing with simple conceptual tools in comment like clarity. If we included that `CachedAPI` to 'simplify' things, we would likely end up doing the opposite and forgetting the specifics: "What exactly does a `CachedAPI` do again? Does it write to the database on every GET?"

# Conclusion
Overall, the key takeaways behind this heuristic and this post in general are as follows:
- Understand the difference between a generic instrument, a purpose built performer, and a composer that does more then one thing, and write them accordingly.
- Strongly prefer a solutions with generic instruments over a purpose built performers. It will likely help you out in other parts of the code, while making the original easier to test and understand.
- Identify composers that are doing more than one thing, and give them the respect they deserve by breaking up their composite parts. 
- Don't break up code simply to reduce repetition or length. Don't extract more performers out of a long performer, it likely won't help. Be conscious about what the code is doing, and why you're breaking it apart the way you are.

Like I said, that 80+ line first step from example 2 was the only backend function in my entire project to nit fit on my 27' monitor. In an object-oriented world, there are very few circumstances where any block of code should be that large.

Anyway, I hope you enjoyed this long voyage through this silly little heuristic that I have been using to write my silly little apps (and to earn a living, I guess). I would love to hear your thoughts via an email response at the link at the bottom of this page. Especially if you disagree, I would love to learn from you!

So go out there and write your own software orchestra to play your favorite tune, whether that happens to be the pure joy of creation, the pride and fulfillment of creating something useful, or even if your just looking for that sweet, sweet green.

---
# Challenge
If you like this heuristic, yet your not sure if you fully get it, I have some practice for you. Do some LeetCode problems that require a specific data structure, such as example 1 from this post. But be sure to create an instrument to represent that data structure that is completely context independent, and then interface with it in your solution. Bonus points if you compose two together, but would likely be a tougher problem, so don't rush into that from the start. I'd be willing to bet your understanding of the underlying data structure will be even stronger than it would with a performer solution, and you'll also be practicing writing maintainable code for use in large projects.

If your looking for a good place to start, check out [Ashish Pratap Singh's](https://blog.algomaster.io/) awesome blog posts on [15 LeetCode Patterns](https://blog.algomaster.io/p/15-leetcode-patterns) and [20 Dynamic Programming LeetCode Patterns](https://blog.algomaster.io/p/20-patterns-to-master-dynamic-programming), and solve the linked problems. By knowing the correct pattern to use from the start, you can focus all your energy into the interface itself and making the instrument generic.




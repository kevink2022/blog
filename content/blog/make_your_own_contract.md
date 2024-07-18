---
author: Kevin Kelly
title: Make your own contract.
date: 2024-07-11
description: Don't try and force a dictionary into set sized shoes.
tags:
  - software
  - software-design
  - software-complexity
categories: 
series: 
aliases: 
draft: false
---

I want to talk about a specific problem I just worked on, and how two logically identical solutions can make a huge difference in taming the complexity of software. 

Boiling it down to the essentials, we have a map, or dictionary, of `ID` objects to `Tag` objects, called `tagMap`. This map is built with the following three operations:
- `add(id: ID, tags: Set<Tag>)` 
	- add a new `ID` with its associated `Tag` objects  (`ID` is guaranteed to *not exist* in `tagMap`)
- `update(id: ID, tags: Set<Tag>)`
	- update the tags for an `ID` (`ID` is guaranteed to exist in `tagMap`)
- `delete(id: ID)`
	- delete the `ID` from the map (`ID` is guaranteed to exist in `tagMap`)

How would you keep track of a *set* of all of the tags in use *for use elsewhere in the program?*

If you really want to leetcode this, here's the code so far:

``` swift
class TagManager

	var tagMap: [ID: Set<Tag>] = [:]

	func add(id: ID, tags: Set<Tag>) {
		tagMap[id] = tags
	}

	func delete(id: ID) {
		tagMap[id] = nil
	}
	
	func update(id: ID, tags: Set<Tag>) {
		let oldTags = tagMap[id]
		delete(id: id, tags: oldTags)
		add(id: id, tags: tags)
	}
}
```

---

I landed on using a new dictionary, mapping a `Tag` to the number of times that tag occurs in the `tagMap`. With that, we can simply add each tag to the dictionary on an add/update, and remove each tag on an update/remove. 

``` swift
class TagManager {

	var tagMap: [ID: Set<Tag>] = [:]
	var tagCounts: [Tag: Int] = [:] // NEW
					
	func add(id: ID, tags: Set<Tag>) {
		tagMap[id] = tags

		// NEW
		for tag in tags {
			let count = tagCounts[tag] ?? 0 
			tagCounts[tag] = count + 1
		}
	}

	func delete(id: ID) {
		let tagsToDelete = tagMap[id] ?? [] // NEW
		tagMap[id] = nil

		// NEW
		for tag in tagsToDelete {
			let count = tagCounts[tag] ?? 0 

			if count == 1 { 
				tagCounts[tag] = nil
			} else {
				tagCounts[tag] = count + 1
			}
		}
	}
}
```

This *works*, but look back at what I emphasized in the question earlier: a set for use elsewhere in the program. Our conceptual model of `tagsInUse` is a set, but we have a dictionary. We can easily solve this in the public API:

``` swift
private	var tagCounts: [Tag: Int] = [:]
public var tagsInUse: Set<Tag> { Set(tagCounts.keys) }
```

But why settle for fixing this in the public API? Bringing that concise conceptual model down into our own logic will help us write code thats more intentional and easier to understand. I think a good way to demonstrate the communication weakness this current solution presents is through the telephone game - explaining your logic to a colleague over the phone.

> We keep track of the *set* `tagsInUse` using a *dictionary* called `tagCounts`, where we keep count of the number of instances of that tag. So when we add a tag to the *set* `tagsInUse` for the first time, we add it to the *dictionary* `tagCounts` with a count of one. If we add it again, we update it count to 2. Now, if we remove it once, we just drop it count to 1, its still in our *set* `tagsInUse`, which is derived from the keys of `tagCounts`. Only when we remove it again, do we actually remove it from the *set* by removing the key from the *dictionary,* since the count reached zero.

This makes sense, and can probably be followed, but again I emphasized the use of *set* and *dictionary* to point out the weird tension here between the conceptual model and the implementation. We can't use a set, because that data structure won't actually keep track of the information the way we need. But we're sort of *breaking the semantic contract* of a dictionary as well. We don't actually care about the specific count, we just care if the tag is one of the keys. 

I think this idea, of a *using a dictionary* to *track a set* over multiple inserts and removals, is a distinct, generalizable conceptual model, separate from a regular set or a dictionary. So, lets create a new semantic contract: a **Counted Set.** 

``` swift
class CountedSet<Element> {
	private var storage: [Element: Int]

	mutating func insert(_ element: Element) {
        if let count = storage[element] {
            storage[element] = count + 1
        } else {
            storage[element] = 1
        }
    }

	mutating func remove(_ element: Element) {
        if let count = storage[element] {
            if count <= 1 { storage[element] = nil }
            else { storage[element] = count - 1 }
        }
    }

	/* Insert/Remove contentsOf Sets/Arrays */
}
```

And our code becomes trivially simple:

``` swift
class TagManager
	var tagMap: [ID: Set<Tag>] = [:]
	var tagsInUse: CountedSet<Tag> 

	func add(id: ID, tags: Set<Tag>) {
		tagMap[id] = tags
		tagsInUse.insert(contentsOf: tags)
	}

	func delete(id: ID) {
		tagMap[id] = nil
		tagsInUse.remove(contentsOf: tags)
	}
}
```

And so does our explanation:

> We keep track of the *set* `tagsInUse` using a counted *set*.

This new explanation relies on the readers knowing what a counted set is. But the counted set can be easily explained and understood outside of the context of the problem. And with that understanding, the explanation of the logic becomes explicit *in* the code through the semantics of the counted set and its documentation, while the original implementation (which is logically the exact same) has no explanation at all, requiring the inferior solution of comments.

On top of that, reifying the conceptual model of the counted set chunks it into a single idea into our heads, freeing up room in our limited working memory for the rest of the problem. In the original solution, the sets of `Tag` and the `tagMap` dictionary of `ID` to `Set<Tag>`, which don't break the semantic contracts of the their data structures, are easily thought of as single ideas. They don't clog our working memory with extra details. But the `tagCount` dictionary isn't just a dictionary, it's a dictionary with extra rules, where a key's values aren't changed arbitrarily. An implicit contract, that needs to be implemented correctly in multiple places. Multiple things to keep track of in our heads.

But once we go ahead and make the contract explicit, putting a label on it, our brains can chunk it and treat it just like another set or dictionary. Now, alongside our `tagMap`, our `tagsInUse` is just another simple data structure we use, with an explicit contract and conceptual model, cleanly labeled in our working memory. 

I think this is such an essential concept, because we often don't even get the privilege of the telephone game. Our code may be read by someone else, in a different timezone, years after we wrote it. So instead of getting to explain to them the conceptual model we had when writing it, all of that intentionality has to be documented in the code. 

Computer Science is a mathematical discipline: we're looking for a *correct* solutions. In my opinion, Software Engineering is much more of a *communication* discipline: there may be many correct solutions, but its vital to consider how your solution communicates your conceptual model to future maintainers (including yourself) in order to keep the software complexity monster in check.
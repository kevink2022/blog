---
author: Kevin Kelly
title: Boomic's 'Worry Free' Database as a Value.
date: 2024-08-17
description: A Datomic Inspired Database in Swift that allows users to undo any errors.
tags:
  - software
  - software-design
categories: 
series: 
aliases: 
draft: false
---
Earlier this week, I was importing songs onto [Boomic Music](https://github.com/kevink2022/Boomic), a music app I've been working on. Not some test files, but some real songs: I've been using the app daily for a few months now, despite it still being in development. It was only a small batch of 41 songs, so I expected the import to run instantly. 

However, it took *slightly* longer than instantly, long enough for me to catch the number of songs being imported. *Was that three digits?* I went and checked the logs, and to my horror, I saw it had imported **520 songs.** I instantly realized what happened: it reimported songs that I had hidden from my library by an organization tool I was working on. 

**UGH.** It was bound to happen eventually. At least it didn't delete anything. Time to go write a temporary function to run some ad hoc SQL and delete those specific songs. Not the end of the world, but there are so many other things I'd rather be working on right now. Now I'm in a rotten mood. At least I would be. If I hadn't been lying to you this whole time. 

In reality, I just kinda shrugged. The logs I checked weren't debug logs, it was the *Library Change History,* built right into the app. I checked the details of the song import transaction and saw which songs were reimported, which clued me in to what went wrong. I simply rolled back the import, and my library was in the exact state it had been 2 minutes ago. I disabled the troublesome setting, hit import again, and the correct 41 songs were imported. I logged the issue and continued my day with a smile on my face. 

When I started Boomic, the data management was what scared me the most. I had no delusions about my skill set: I am young, inexperienced, and most importantly, human. Of course I was going to make mistakes. But data management mistakes are the *worst*. One small error and half the library I'd spent hours organizing gets wiped. I was terrified enough of doing that to myself, let alone doing it to any of my friends who were generous enough to offer to help me test it.

So, when I heard about the [Datomic database](https://www.datomic.com/), and saw Rich Hickey explain it at a high level in [*Database as a Value*](https://www.youtube.com/watch?v=EKdV1IgAaFc), I knew I wanted it for Boomic. The whole database as an immutable value that could be passed into a function. Where any of those past values, from *any* point in time, could be retrieved, allowing any mistake to be undone. How cool is that? No need to worry deleting my friends of library, because it could just be rolled back!

One small issue: It's closed source ([but free!](https://blog.datomic.com/2023/04/datomic-is-free.html)) and runs in the JVM. I wanted to keep the scope of this app on device, at least for the initial release, so I could focus on a single code base as I learned application architecture. So, if I want the comfort of the database as a value, I'll have to build it myself. How hard could that be?

# Free Observation: The Epochal Time Model.
The first foundational idea from Datomic (and Clojure in general) we need to understand is the **Epochal Time Model,** where the *state* of an *identity* is moved from immutable *value* to immutable *value* over time. The specific definitions of the terms state, identity, and value come from Rich Hickey's 2009 [*Are We There Yet?*](https://www.youtube.com/watch?v=ScEPu1cs4l0&t=2832s), but the gist is:

- **Identity:** The label to which we refer to something. For example, the "My Laptop Brand."
- **Value**: A value of an identity. For example, at one point, I owned as "ASUS" laptop.
- **State:** A *value* at a *time,* usually associated to an identity. So while the value of the identity "My Laptop Brand" was "ASUS" a few years ago, the current state is "Apple."

So bringing it to a pseudo-code example with a 'Tech Tracker' app, we can create a variable `laptopBrand` to hold the *state* of the *identity* of "My Laptop Brand" And then we can assign the *values* of "ASUS" and "Apple," changing the state of the identity when I get a new laptop.

There is a huge drawback to using a simple variable that we often accept as inevitable: we share state. To read the state of an identity in a multithreaded environment, we usually need to block all other threads to ensure it doesn't change and cause some kind of race condition. Imagine if I couldn't post this [incredibly cute picture of my dog](/blog/images/tito.jpg) on Instagram because someone was looking at my profile at the time, and to do so had to 'lock' it. Or if only one person could look at Taylor Swift's Instagram at a time, because they had to lock it? She would never get to post! But how else are we supposed to avoid race conditions?

The Epochal Time Model directly addresses this issue. As shown in Figure 1, each value of the identity over time is a separate reference. Observers can just grab the current value, representing the current state, and not worry about it being changed by some other thread as it's immutable. The thread running some long operation on V<sub>2</sub> doesn't need to block the rest of the program from moving forward, it can continue with slightly stale data. And if the data truly cannot be stale, we can subscribe to a message queue and cancel when a new value gets published.

![The Epochal Time Model](/blog/images/epochal_time.jpg)

Figure 1. The Epochal Time Model

The first objection many people have to this approach is the duplication of data, especially when dealing with collections. Assuming V is a large array, we now have 3 active values of V, (assuming V<sub>1</sub> has been garbage collected in a reference counting environment). This problem is mitigated in Clojure/Datomic with the use of [persistent data structures](https://hypirion.com/musings/understanding-persistent-vector-pt-1), though it can be a problem with Swift's copy-on-write behavior. 

However, with the scale I was planning on this project, and knowing I could always tune it based on persistent data-structure concepts, I wasn't too worried. I have come up with a little motto when doing this project: "Compute is free, until it's not." If I didn't notice any slowdown, and it wasn't a memory bottleneck, I considered it was free, *for now*. And when some rough calculations/tests on a 10k song library (which would probably be >100gb of audio) showed it likely wouldn't be, I decided to move forward with it.

# Remembering the Past: Reifying Change.
With our newfound knowledge of the Epochal Time Model, we can start building a database on the same foundations of Datomic: turning the entire database into an epochal value. So do we just save the history of values? Even with persistent data structures, we would still be storing a lot of duplicate data.

Turning back to Datomic and Rich Hickeys high level talks, we can paint a general picture of the Datomic Architecture:
1. All writing is done through `Transactor` servers, which create and publish `Transactions`. `Transactions` are made up of `Assertions` and `Retractions`, which **reify change**, or describe exactly what is changing between two values of the database. 
2. Any `Application` server can use pure functions to apply new transactions (live index) to a value of the database from the `Storage` server (cache) to get newer values to query against, which are each immutable. 
3. The actual data on the `Storage` servers uses a [data structure with memory](https://www.geeksforgeeks.org/persistent-data-structures/) (also called persistent data structures, but different then in Clojure) to queue on past values of the database.  The analogy Rich uses is the rings of a tree, with each transaction being a new ring. You can query any ring, which contains all the data up until that ring.

![Datomic Architecture](/blog/images/datomic_architecture.png)
Figure 2. Datomic Architecture

This architecture gives Datomic ACID properties while still allowing horizontal read scaling of Application and Storage servers, although write vertically scales with Transactor servers. The storage server is where the real magic happens, storing every single value of the database in one large structure, and is what allows [Nubank](https://nubank.com.br/en/), the biggest bank is Brazil, to deploy Datomic at scale.

Boomic, on the other hand, aims to be the biggest music player app on my tiny little phone. We're talking tens of thousands of records, not billions. So we could break the rules a little bit, and:
- Only store the `Transaction` history.
- Rebuild values of the database using the `Transaction` history (and maybe snapshots). 

This allows us to build a much simpler version without losing the ability to retrieve and any past value of the database.

But we still need to "reify change" in the transactions. How do we go about that? Datomic uses generic `Assertions` and `Retractions`, which as the name suggests assert and retract new values of identities, changing their state. So, if `laptopBrand = "ASUS"` at time t<sub>0</sub>, an assertion at time t<sub>1</sub> might say `laptopBrand = "Apple"`. Then, at time t<sub>2</sub>, we can retract that value. So if we query the value of the database between times t<sub>1</sub> and t<sub>2</sub>, `laptopBrand` would be `"Apple"`, but outside of that, it would be `"ASUS"`.

Could we do something similar in Swift? We can, but the `Assertion/Retraction` system really synergizes well with Clojure/Datomic's dynamic typing, with their general approach to data being 'everything is a map.' There isn't a compiler to stop you from asserting that my `headphoneBrand = "AKG"`, even though `headphoneBrand` was not previously tracked by the database. And when we say `laptopBrand`, the database will assume we know what we're talking about.

There are a lot more guard rails in the strongly typed world of Swift. It would be nice to say something like "Assert `albumA.artist` is `"Maryline Léonard"`, but Swift doesn't know how to store just that information without storing the whole album object.

The brute force method would be to just store a whole copy of the new data type whenever we changed something. "Assert `albumA` is `newAlbumA`" where everything in `newAlbumA` is the same as `albumA`, except the artist. But while I consider the extra memory of copy on write 'free for now,' I don't have the same mindset about gobbling up storage that way. I want to store the bare minimum: exactly what changed, and nothing more. 

What we could do instead is create new data types to store our updates. "Assert `albumA` is `albumA.apply(albumUpdate)`." Here, we could just store the `AlbumUpdate`, which is a mirror of the `Album` type, but where everything is optional. So when stored as JSON, the storage used would be proportional to the size of the actual change.

Which is what I ended up doing, and it took some serious boilerplate. I mean, it's technically not boilerplate, but making a `TypeUpdate` that mirrors each `Type` I wanted to use sure felt like it. And that had to be done for each type I wanted to 'transact' with in this way.

With these update types, we can go ahead and create a generic `Assertion` with three basic types:
- Add: Add a new `Song`, `Album`, `Artist`, etc.
- Update: Update a  `Song`, `Album`, `Artist`, etc. with a  `SongUpdate`, `AlbumUpdate`, `ArtistUpdate`, etc<sub>(update)</sub>.
- Delete: Delete a `Song`, `Album`, `Artist`, etc.

Finally, with these assertions, we have reified change without duplicating too much data, and we can build a functional database: We can take a value of the database, apply a transaction with `Assertions` to it, and get a new value! All purely functional and easy to test. But how do we manage the storage of these transactions, and build new database values on command?
# A 'Swift' Transactor
Lastly, we need to build the `Transactor` class to represent the epochal time model and save transactions. It should do the following:
- Save the `Transaction` History.
- Publish (or `Post`) new values.
- Apply each `Transaction` to the previous `Post` value to generate new `Post` values. 
- Show the `Transaction` history.
- Rebuild and publish any `Post` from before or after any `Transaction`.

The `Transactor` has two associated generic types, the `Transaction` and the `Post`. In Boomic, that's:
- The `Transaction`, called `LibraryTransaction`, holds the `Assertions`. 
- The `Post`, called `DataBasis`, is a value of the database to be queried against. 

Figure 2 shows us what that might look like in our epochal time model, with P<sub>n</sub> as `Posts`, f<sub>n</sub> as functions that move the `Post` forward, and T<sub>n</sub> as the `Transactions` that are saved.

![Conceptual Transactor with the Epochal Model.](/blog/images/transactor_epochal_concept.svg)

Figure 3. A conceptual transactor.

But this doesn't paint the whole picture. To build a `Post`, the `Transactions` need to have the complete context, including the function f<sub>n</sub> that generated the new post from the old one.

But what's the point in saving the transactions if we need to save each of the functions anyway? The whole idea behind the `Transaction` is that it has all the information needed to describe the change between one state of the `Post` and the next. So, we can define a single `commit()` function that utilizes a `Transaction` to create the next value of the post. Now, we have a transactor that looks like this:

![Transactor with the generate transaction and apply transaction steps separated.](/blog/images/transactor_epochal.svg)

Figure 4. A transactor with the generate half and the commit half split.

The new transactor has split the process into two distinct halves: The first in red, for generating new transactions. This can be done with any function that creates a `Transaction` from a `Post`, and doesn't need to be stored. 

Then we have a second half in blue, a predefined way of generating any value (P<sub>n</sub>) as long as we have a base value (P<sub>0</sub>) and the transaction history (\{T<sub>1</sub>...T<sub>n</sub>}). Notice how you can completely remove the red process once you have the transactions.

So, on the initialization of the transactor, we have to provide:
- The Base `Post`: **P**<sub>0</sub>
- The `Transaction` `commit()` function f: P<sub>n</sub> = **f**(P<sub>n-1</sub>, T<sub>n</sub>)

Then, we can use any g functions we define to generate new transactions: 
- Generate new transaction function: T<sub>n</sub> = **g**<sub>n</sub>(P<sub>n-1</sub>)

So, were done then? Not so fast. We probably should design transactor to be completely asynchronous, both for async multithreading in generating/committing large transactions, and to accommodate remote storage options. 

But when we do that, rebuilding a `Post` becomes very slow: It only takes a few dozen transactions for it to become noticeable by an end user. Our debt has caught up to us: this compute is certainly longer free, and now we need to find a way out. We might jump to snapshots, but if we're taking and storing a snapshot every dozen transactions, is there even a point? If only we could shrink the number of steps it takes to rebuild...

Luckily, there is a way to can shrink it down to just two steps: Flatten all the transactions into one combined transaction, and the commit it to the base post. Think about it, if we add a song, then update that same song later, wouldn't that be equivalent to adding the song with the update already applied? And if we delete that song, there's no need to even add it in the first place.

![The flatten function, that combines each transaction and only applies once.](/blog/images/transactor_epochal_rebuild.svg)
Figure 5. Rebuilding P3 from a singled Flattened Transaction.

So to rebuild any `Post` P<sub>n</sub> in two steps, we:
- Create the combined transaction: T<sub>{1...n}</sub> = h( \{T<sub>1</sub>...T<sub>n</sub>} )
- Commit the combined transaction: P<sub>n</sub> = f(P<sub>0</sub>, T<sub>{1...n}</sub>)

And now we've hit all the general concepts of the transactor as it is now, storing data and fixes mistakes in Boomic. A `Post`, or "database as a value" is called a `DataBasis`, because passing around a database felt off. The `Transaction` is a called `LibraryTransaction`, which holds a message about the transaction, the significance of the transaction, and the set of `Assertions` to be applied.

And all the pure functions, both the **g** functions that generate new transactions and the **f** function that commits existing ones, contained within the `BasisResolver`.

# The Code
So what does it look like in action? First we initialize:

1. Declare the `Transactor` with:
	- The key (for persistence).
	- The base `Post`: P<sub>0</sub>
	- The transaction function: f(P<sub>n-1</sub>, T<sub>n</sub>) = P<sub>n</sub>
	- The flatten function: h( \{T<sub>0</sub>...T<sub>n</sub>} ) = T<sub>{0...n}</sub>
``` swift
let transactor = Transactor<LibraryTransaction, DataBasis>(
	key: "BasisTransactor"
	, basePost: DataBasis.empty
	, transactionCommit: { transaction, basis in
		BasisResolver(basis).commit(transaction)
	}
	, flattenTransactions: { transactions in
		transactions.flatten() // extension on array
	}
)
```

2. Subscribe to the transactor publisher to receive new posts.
``` swift
transactor.publisher
	.sink { [weak self] basis in
		self?.basis = basis
	}
	.store(in: &cancellables)
```

Then, we have our two options of committing transactions:

3. Commit an existing transaction by calling the commit function: f(P<sub>n-1</sub>, T<sub>n</sub>) = P<sub>n</sub>.  This isn't done anywhere in Boomic, it is only done internally by the `Transactor` to rebuild values, but the option is there.
``` swift
await transactor.commit(transaction)
```

4. Generate a transaction based on the current basis: g<sub>n</sub>(P<sub>n-1</sub>) = T<sub>n</sub>. The commit function above is automatically called on the `Transaction` generated by `addSongs()`. 
``` swift
await transaction.generate { basis in
	await BasisResolver(basis).addSongs(newSongs)
}
```

And now we have access to the entire history, and ability to roll back to any point.

5. View transaction history: \{T<sub>0</sub>...T<sub>n</sub>}
``` swift
let transactions = await transactor.history()
```

6. And then rollback to before, or after a transaction (publisher will post when rollback completes):
``` swift
await transactor.rollback(toBefore: mistakeTransaction)
```

# Conclusion
And that is how I am currently handling library data in Boomic. It was a lot of work, but I learned a lot, and it has already saved me from writing ad hoc SQL once. There is a lot more I could do, such as applying but not committing a transaction until after we see what the post would be, or seeing a rollback post before actually deleting the newer transactions. I'm sure I'll implement those in the future, but the foundation is pretty much here, and I thought it might make for some interesting reading.

I hope you enjoyed and learned something new. I would love to hear from you: constructive criticism, ideas for improvement, or even just how your day is going. You can respond to this post via email at the link below, or just shoot me a message on [LinkedIn](https://www.linkedin.com/in/kevink2019/).

Thank you for reading, have a wonderful day!
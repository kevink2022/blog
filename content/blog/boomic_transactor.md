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

However, it took *slightly* longer then instantly, long enough for me to catch the number of songs being imported. *Was that three digits?* I went and checked the logs, and to my horror, I saw it had imported **520 songs.** I instantly realized what happened: it reimported songs that I had hidden from my library by an organization tool I was working on. 

**UGH.** It was bound to happen eventually. At least it didn't delete anything. Time to go write a temporary function to run some ad hoc SQL and delete those specific songs. Not the end of the world, but there's so many other things I'd rather be working on right now. Now I'm in a rotten mood. At least I would be. If I hadn't been lying to you this whole time. 

In reality, I just kinda shrugged. The logs I checked weren't debug logs, it was the *Library Change History,* built right into the app. I checked the details of the song import transaction and saw which songs were reimported, which clued me in to what went wrong. I simply rolled back the import, and my library was in the exact state it had been 2 minutes ago. I disabled the troublesome setting and hit import again. The correct 41 songs were imported, I logged the issue, and I continued with my day with a smile on my face. 

When I started Boomic, the data management was what scared me the most. I had no delusions about my skillset: I am young, inexperienced, and most importantly, human. Of course I was going to make mistakes. But data management mistakes are the *worse*. One small error and half the library I spent hours organizing gets wiped. I was terrified enough of doing that to myself, let along doing it to any of the friends who were generous enough to offer to help me test it.

So, when I heard about the [Datomic database](https://www.datomic.com/), and saw Rich Hickey explain it at a high level in [*Database as a Value*](https://www.youtube.com/watch?v=EKdV1IgAaFc), I knew I wanted it for Boomic. A database as an immutable value that could be passed into a function. Where any of those past values, from *any* point in time, could be retrieved, allowing any mistake to be undone. How cool is that? No need to worry deleting my friends of library, because it could just be rolled back!

One small issue: Its closed source, ([but free!](https://blog.datomic.com/2023/04/datomic-is-free.html)), and runs in the JVM. I wanted to keep the scope of this app on device, at least for the initial version, so I could focus on a single code base as I learned application architecture. So, if I want the comfort of the database as a value, I'll have to build it myself. How hard could that be?

# Free Observation: The Epochal Time Model.
The first foundational idea from Datomic (and Clojure in general) we need to understand is the **Epochal Time Model,** where the *state* of an *identity* is moved from immutable *value* to immutable *value* over time. The specific definitions of the terms state, identity, and value come from Rich Hickey's 2009 [*Are We There Yet?*](https://www.youtube.com/watch?v=ScEPu1cs4l0&t=2832s), but the gist is:

- **Identity:** The label to which we refer to something. For example, the "My Laptop Brand."
- **Value**: A value of an identity. For example, at one point, I owned as "ASUS" laptop.
- **State:** A *value* at a *time,* usually associated to an identity. So while the value of the identity "My Laptop Brand" was "ASUS" a few years ago, the current state is "Apple."

So bringing it back to a pseudo code example with a 'Tech Tracker' app, we can create a variable `laptopBrand` to hold the *state* of the *identity* of "My Laptop Brand" And then we can assign the *values* of "ASUS" and "Apple," changing the state of the identity when I get a new laptop.

There is a huge drawback, often accepted as inevitable, to using a simple variable: we share state. To read a the state of an identity in a multithreaded environment, we usually need to block all other threads to ensure it doesn't change and cause some kind of race condition. Imagine if I couldn't post this [incredibly cute picture of my dog](tito.jpg) on Instagram because someone was looking at my profile at the time, and to do so had to 'lock' it. Celebrities would never get to post! But how else are we supposed to avoid race conditions?

The Epochal Time Model directly addresses this issue. As shown in Figure 1, each value of the identity over time is a separate reference, so observers can just grab the current reference, representing the current state, and not worry about it being changed by some other thread. The thread running some long operation on V<sub>2</sub> doesn't need to block the rest of the program from moving forward, it can continue with slightly stale data. And if the data truly cannot be stale, it can just cancel, or run again when its finished.

![Original Database with coupled file interface](/blog/images/epochal_time.jpg)
Figure 1. The Epochal Time Model

The first objection many people have to this approach is the duplication of data. We now have 3 active values of V, assuming V<sub>1</sub> has been garbage collected in a reference counting environment. This is largely mitigated in Clojure with the use of [persistent data structures](https://hypirion.com/musings/understanding-persistent-vector-pt-1), though it can be a problem with Swift's copy-on-write behavior. 

However, with the scale I was planning on this project, and knowing I could always tune it based on persistent data-structure principles, I wasn't too worried. I have come up with a little motto when doing this project: "Compute is free, until its not." If I didn't notice any slowdown, and it wasn't a memory bottleneck, it was free for now. And when some rough calculations/tests on a 10k song library (which would probably be >100gb of audio) showed it likely wouldn't be, I decided to move forward with it.

# Remembering the Past: Reifying Change.
With our newfound knowledge of the Epochal Time Model, we can start building a database on the same foundations of Datomic: turning the entire database into an epochal value. So do we just save the history of values? Even with persistent data structures, we would still be storing a lot of duplicate data.

Turning back to Datomic and Rich Hickeys high level talks, we can paint a general picture of the Datomic Architecture:
1. All writing is done through `Transactor` servers, which create and publish `Transactions`. `Transactions` are made up of `Assertions` and `Retractions`, which **reify change**, or describe exactly what is changing between two values of the database. 
2. Any `Application` server can use pure functions to apply new transactions to get a new value of the database, which is immutable. These can be tested locally before they're actually committed. 
3. The actual data on the `Storage` servers uses a [data structure with memory](https://www.geeksforgeeks.org/persistent-data-structures/) (also called persistent data structures, but different). The analogy Rich uses is the rings of a tree, with each transaction being a new ring. You can query any ring, which contains all the data up until that ring.

This architecture gives Datomic ACID properties while still allowing horizontal read scaling (although write vertically scales). The third point is where the real magic happens, and what allows [Nubank](https://nubank.com.br/en/), the biggest bank is Brazil, to deploy Datomic at scale.

Boomic, on the other hand, aims to be the biggest music player app on my tiny little phone. We're talking tens of thousands of records, not billions. So I could break the rules a little bit, and I did:
- Only store the `Transactions` history.
- Rebuild values on request using `Transaction` history (and maybe snapshots). 

So, how do we "reify change?" Well, Datomic uses generic `Assertions` and `Retractions`, which, as the name suggests assert and retract new values of identities. So, if `laptopBrand = "ASUS"` at time A, an assertion at time B might say `laptopBrand = "Apple"`. Then, at time C, we can retract that value. So if we query the value of the database between times B and C, `laptopBrand` would be "Apple", but outside of that, it would be "ASUS."

Could we do something similar in Swift? We can, but the `Assertion/Retraction` system really synergizes well with Clojure/Datomic's dynamic typing, with their general approach to data being 'everything is a map.' 

In the strongly typed world of Swift, we run into issues. We could just store a whole copy of the new data type whenever we changed something, but while memory wasn't an issue, I was a bit worried about using too much storage that way. I wanted to store the bare minimum: exactly what changed, and nothing more. Which took some serious boilerplate. I mean, its technically functional, but making a `SongUpdate` that mirrors the `Song` type except everything is optional certainly feels like boilerplate. And that had to be done for each type I wanted to 'transact' with.

With these update types, I went and created a generic `Assertion` with three basic types:
- Add: Add a new `Song`, `Album`, `Artist`, etc.
- Update: Update a  `Song`, `Album`, `Artist`, etc. with a  `SongUpdate`, `AlbumUpdate`, `ArtistUpdate`, etcUpdate. 
- Delete: Delete a `Song`, `Album`, `Artist`, etc.

Finally, with these assertions, I had a fully functional database: I could take a value of the database, apply a transaction of `Assertions` to it, and get a new value! But how do we manage the storage of these transactions, and build new database values on command?

# A 'Swift' Transactor
Finally, we get to the implementation. To represent the epochal time model and save transactions, we can build a generic `Transactor` class, which has the following requirements:
- Save `Transaction`s.
- Publish (or `Post`) new values.
- Apply each `Transaction` to the previous `Post` to generate new `Post` values. 
- Show a `Transaction` history.
- Rebuild and publish any `Post` from any point in history using the `Transactions`.

The `Transactor` has two associated generic types, the `Transaction` and the `Post`. For the Boomic Database, the `Transaction` held the `Assertions`, and the `Post` was the value of the database to be queried against. Figure 2 shows us what that might look like in our epochal time model, with P<sub>n</sub> as `Posts`, f<sub>n</sub> as functions that move the `Post` forward, and T<sub>n</sub> as the `Transactions` that are saved.

![Conceptual Transactor with the Epochal Model.](/blog/images/transactor_epochal_concept.svg)
Figure 2. A conceptual transactor.

But this doesn't paint the whole picture. To rebuild the a `Post`, the `Transactions` need to have the complete context. All of the information, including the function that generates a new post from an old one and a transaction.

But what's the point in saving the transactions if we need to save each of the functions anyway? The whole idea behind the `Transaction` is that it has all the information needed to describe the change between one state of the `Post` and the next. So, we can define a single `commit()` function that commits a `Transaction` onto a value of the post.

![Transactor with the generate transaction and apply transaction steps separated.](/blog/images/transactor_epochal.svg)
Figure 3. A transactor with the generate half and the commit half split.

So now we have split the process into two distinct halves: The first in red, for generating new transactions, which can be done in many different ways that don't need to be remembered. Then a second half in blue, a predefined way of generating any value (P<sub>n</sub>) as long as we have a base value (P<sub>0</sub>) and the transaction history (T<sub>1...n</sub>). Notice how you can completely remove the red generate process once you have the transactions.

So, on the initialization of the transactor we have to provide:
- The Base `Post`: P<sub>0</sub>
- The `Transaction` `commit()` function f: P<sub>n</sub> = f(P<sub>n-1</sub>, T<sub>n</sub>)

Then, we can use any g functions we define to generate new transactions: T<sub>n</sub> = g<sub>n</sub>(P<sub>n-1</sub>)

So, were done then! Well, as usually, there is one more thing. The transactor is designed to be completely asynchronous, both to allow for multithreading in closures used to generate and apply transactions, and for potential remote storage options. With that, the rebuilding is functional but it's very slow. It only took a few dozen transactions for it to become noticeable by an end user. Our debt has caught up to us: the compute is no longer free, and we need to find a way out.

We could run away from the issue and use a synchronous commit function. But if we want to keep the benefits of `asyc`, we need to find a way to speed it up. 

One method would could use is flattening all transactions into one. If we're add a song, then updating that same song later, wouldn't that be equivalent to adding the song with the update already applied? 

![The flatten function, that combines each transaction and only applies once.](/blog/images/transactor_epochal_rebuild.svg)
Figure 4. The Flatten Function

Now we've made it to how it works in Boomic. A `Post`, or "database as a value" is called a `DataBasis`, because passing around a database felt weird.. The `Transaction` is a called `LibraryTransaction`, which holds a message about the the change, the significance of a change, as well as the set of `Assertions`, which are what is actually applied to the `DataBasis` to make changes. All these changes, both the g functions that generate new transactions and the f function that commits existing ones, are done with through the `BasisResolver` using pure functions.

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


Then, we have out two options of committing transactions:

3. Commit an existing transaction by calling the commit function: f(P<sub>n-1</sub>, T<sub>n</sub>) = P<sub>n</sub>.  This isn't done anywhere in Boomic, it is only done internally by the `Transactor` to rebuild values, but the option is there.
``` swift
await transactor.commit(transaction)
```

4. Generate a transaction based on the current basis: g<sub>n</sub>(P<sub>n-1</sub>) = T<sub>n</sub>. The commit function above is automatically called on the new function. 
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
And that is how I am currently handling library data in Boomic. It was a lot of work, but I learned a lot, and it has already saved me from writing ad hoc SQL once. There is a lot more I could do, such as applying but not committing a transaction until after we see what the post would be, or seeing a rollback post before actually deleting the newer transactions. I'm sure I'll implement those in the future, but the foundation is pretty much there, and I thought it might make for some interesting reading.

I hope you enjoyed and learned something new. I would love to hear from you: constructive criticism, ideas for improvement, or even just how your day is going. You can respond to this post via email at the link below, or just shoot me a message on [LinkedIn](https://www.linkedin.com/in/kevink2019/).

Thank you for reading, have a wonderful day!
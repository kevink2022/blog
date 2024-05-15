---
author: Kevin Kelly
title: Do you understand it, or do you just know it?
date: 2024-05-13
description: How my entire understanding of what learning is was flipped on its head.
tags:
  - learning
categories: 
series: 
aliases: 
draft: false
---
This first half of this post is a personal story, adding context to how I came to these conclusions at this point in my life. If you want to jump straight to the difference between knowing and understanding, feel free to skip to [what it means to understand.](#what-it-means-to-understand)

---
### We should really read more.
I was hanging out with US Air Force officers, at the invite of a friend who himself is an officer. We got on the topic about reading, specifically how we all really should read more. Someone made a comment about how the most successful, high ranking officers they knew always seemed to be reading something, and always had a whole bookshelf in their office.

The cynical side of me wanted to say something about how reading all those books is impossible unless you have something driving you more then the vain, ephemeral idea of 'success.' For most people, at least.

At the time, I assumed that drive had to come from some innate interest or curiosity, that had to be there before you ever started learning about the topic. That most of these high ranking officers were, whether they knew it or not, in some way interested in how organizations work and how to effectively lead, and combining that with them being put into a position to lead and/or picking up their first book on the topic, started their pursuit of knowledge. 

I now understand that this line of thinking was an artifact of the 'fixed mindset,' (basically the opposite of a 'growth mindset') that I had just recently recognized was slowing me down. Because I had no issue reading, I've read entire *Song of Ice and Fire* books in less than 2 weeks. But it was a deep interest in the story, the world, and pre-modern societies in general that fueled my rabid reading. Without that underlying interest being awoken, there's no way I would read hundreds of pages or blow through hours of content on something as boring as how to run an organization.

---
### A life of knowing
I was always very, very successful in school with very little effort. I was consistently called the smartest person in the room. While I'm sure people understood that this may create a massive ego problem (and it very much did), what I'm sure what they didn't understand was how they were setting me up to be very conservative in my future endeavors. 

This phenomenon is described in detail in Carol Dweck's writings on [growth mindset](https://ctl.stanford.edu/sites/g/files/sbiybj17446/files/media/file/growth_mindset_handout.pdf), but to sum it up, when people are praised, believe it or not, it makes them happy. Unsurprisingly, this drives them towards behavior directed at continuing to receiving that praise. 

The catch is, when people are praised for their *intrinsic* traits, such as how smart or talented they are, that doesn't encourage them to seek harder challenges, outside their comfort zone. In fact, the opposite usually happens - they seek easier challenges. To them, failure makes them question their own identity and the self-worth that they based on that identity (I can vouch for that). Now, when you praise people for their *extrinsic* trait of *effort*, something they can directly control, well you can imagine how their behavior might differ.

I took pride in my ability to not just cram, but often gave people awful advice to not spend so much time trying to *understand*. You really didn't need to understand why, for example, the derivative of `cos(x)` was `-sin(x)`. That was just another tool in your toolbox that you needed to *know.*

And so I continued through school and graduated college with honors, but always struggled to accomplish anything outside of my coursework. However, I always found excuses for this lack of production, cause I was just too smart to fail, I had just failed to manage my time with my coursework. But then I graduated 6 months early, and took those 6 months off to finally do something with my life.

---
### Beginning to understand
My main goal at the end of those 6 months was to have an app on the iOS App Store. Not a super lofty goal, but I wanted a real, useful app, not just some gimmick. I had taken the Swift UI class at Penn State, so felt that I knew everything I needed to *know* to clean up my rushed final project, which was [Boomic Music](/tags/boomic/). But that was such a mess, as I has rushed to get as many features done during class, didn't spend any time on architecture, and skipped a lot of the hard problems. So I quickly pivoted to something easier: a workout app. Then I pivoted about 3-4 more times over the next few months. 

I can't really pull any lessons out of those 6 months, other than to just focus on one project at a time, because I was just making the same mistakes over and over again. The real problem was that, of course, the 3 credit class I took was nowhere near enough to build an app on the scale I wanted. And I was *really* impatient. When I ran into something I didn't know, I wanted to know it as soon as possible and move on. Just like when I was doing my homework or studying for a test.

Fast forward to the end of 2023, a year since I graduated. I was six months into a very unfulfilling implementation consultant role, and had nothing to put on my resume to give me hope about that changing any time soon. I made a few commitments to myself (such as committing to a growth mindset), and after stumbling upon an article on [10 Things Software Developers Should Learn about Learning](/learning/ten_things_about_learning/), I committed to two more things:

- Actually learning how to architect an app by taking my time and reading as much source code/learning materials as possible
- Focus 100% of my creative energy on finally building Boomic. No more pivoting.

The single text that kick-started my understanding of mobile architectures was Jacob Bartlett's blog post on [Modular Architecture for Apps](https://jacobbartlett.substack.com/p/modular-architecture-for-apps). Following the layered architecture, development was going much better then it had in the past, and I had some newfound confidence. Until I had to throw out 3 weeks of work due to some poor assumptions I made. The familiar fears of of failure and wasted time began to well up inside me.
 
But this time, with a growth mindset, I really reflected on what went wrong, and learned from my mistakes. Oh, and I discovered Rich Hickey.

Over the next few weeks, I went ahead an consumed almost every talk that man has given, and most the keynotes twice. I consistently had better ideas of how to compose my app, both during the talks and in the days after. These presentations, mostly from over a decade ago before the Swift language existed, without a line of source code, gave me so much insight and confidence to write Swift code. It gave me a deeper *understanding* of what I was building. 

This may seem obvious, that there can be wisdom in old material. But for my past self, I just wanted to learn things once. I just wanted to know, so I could do, and move on. And if we're all standing on the shoulders of giants, wouldn't that wisdom get passed to me in the new material anyway?

---
### What it means to understand. 
My aversion to reading older material came from my impatience and desire to just *know*. Just like how I didn't care to know *why* the derivative of `cos(x)` was `-sin(x)`, I just wanted to know the best mobile architecture for the type of app I'm making. Or the best database to use. Or the best *insert solution here* for *insert current problem here*. If someone had made the mistake of giving me a leadership position, if I didn't arrogantly assume I already knew what I was doing, I would buy a brand new bestselling book on that topic. Certainly, the author had read the last bestseller, who's author had read the last bestseller, and so on. And through that magical chain, all of the knowledge of the past would be bestowed upon me, who is certainly smart enough to put it to effective use. 

But I now finally understand why simply knowing is insufficient. Because its simply *impossible* to ever understand and truly grasp complex topics from one perspective. 

Now, if I was in the Matrix and could be plugged directly into that authors brain, maybe then one perspective would be sufficient. But we don't live in the Matrix, and as Marco Giancotti aptly paints with that example, [language is a bottleneck for thought](https://aethermug.com/posts/language-is-a-bottleneck-for-thought). While visual mediums can be orders of magnitude better (a picture is worth a thousand words after all), as of now, we still have no way to communicate the exact intricacies of our mental models of the world to others. Through trying to start writing blog posts, I've learned just how futile that can feel. 

I felt like my world opened up and I finally understood software design after watching Rich Hickey's [Simple Made Easy](/learning/simple_made_easy), but then I watched his [Simplicity Matters](/learning/simplicity_matters), basically the same talk given later, and was surprised how much more I learned. Then I went back and watched the original again, and realized there was so much more I missed.

Listening to Ron Westrum talk about generative organizations on [The Idealcast](/learning/idealcast), again I felt like my world opened up, and I finally understood what made some organizations great (and why work in a bureaucratic organization could feel so ['Sisyphusian'](https://en.wikipedia.org/wiki/Sisyphus)).  But then, I listened to Elisabeth Hendrickson and Steven Spear's first appearances on the show, and not only did it give me a deeper understanding, but it affirmed and built on so much of what Ron Westrum had described in his episodes.

And all the sudden, just like those high ranking officers we all wanted to emulate, I found my bookshelf full of books. 

It was when I started reading the classic [*The Mythical-Man Month*](/learning/the_mythical_man_month) that the difference between knowing and understanding really clicked. I would have never considered reading this book in the past, with it being released all the way back in 1975. I mean c'mon, even the way software was developed in the 90s seemed like a nightmare (git was released in *2005!*), what could this guy know?

But when I was reading it, I found myself constantly making connections to what I had learned previously from Rich Hickey and the Idealcast. Not only was I reading things that affirmed those lessons, but it was reading things that at first seemed to go against my model of the world that made me think the hardest and taught me the most. Things that made me *challenge* my understanding of the world. And at the end, I had a better understanding.

At the end of the day, while it might feel like we think in words, our brains are enormous neural networks with billions of lossy nodes. Even the most prolific thinkers and writers in human history couldn't possibly communicate the complete and deep understanding that they've built up through *any* medium. No matter how many times I listen to everything Rich Hickey has ever said, despite his best efforts, I will never see and understand the world the way he does.

Only through our own constant learning and experience can we hope to build up our own, unique, but just as deep, understanding. As long as we are constantly receiving new inputs, whether through experience, classes, lectures, keynotes, blogs, or the books on our cool bookshelves we originally bought to make us look smart, we can train our neural networks to have as rich and thorough of an understanding as the leaders we admire and want to emulate.

And for all the impatient knowers out there who just want to get stuff done, take it from one of your own. To master a complex topic like software, and to really do it right, you need to learn how to *understand*, or you'll never become that expert who just gets stuff done. After all,

> Beginners Reason, Experts Recognize.
>
> --- [10 Things Software Developers Should Learn about Learning](/learning/ten_things_about_learning/)

And that's why those officers have all those books.

---
### Is there still room for knowing?
Yes, that was not rhetorical. Before we can understand something, we need to know as much as possible. I only made those deep connections reading Mythical Man-Month because I had remembered what Rich Hickey, Elisabeth Hendrickson, and Steven Spear had said. It was those connections between what I knew that lead to understanding. I think the general 'three pillars' model of language learning I acquired while I learned Japanese in college applies here:
1. Learn the words, so you know what the they're saying. 
2. Learn the grammar, so you understand how those words go together.
3. Engage with the language as much as possible.

I needed to know the words and the grammar, so I could engage with the language and build those connections of deep understanding. 

But we really need to be conscious of what we're doing, and why we're doing it. We don't want to waste our time trying to apply an understanding strategy on things we should just know. Like with Japanese words: there was no getting around it, I needed to memorize the words. I needed to just know them, and the grammar rules, so when I heard a sentence, I could make the connections and understand the sentence.

This is a mistake I see my current organization making. Our product is a Goliath of an accounting program. If theres any clear, up-to-date documentation of the inner workings of the system, they haven't been shared with me (even a lot of the *source code* is even hidden from most devs). Instead, they encourage people build a deep understanding of *where* the code is or *where* a fix should be to align with the current system. This may sound confusing, but for various reasons (configuration code in a relational SQL database, classes have cryptic 5-8 letter code names, inter-class calls done through opaque messages), even finding where code is can be difficult. Basically, to make any changes, whether adding something or fixing a bug, the official procedure is to go to that part of the system in the GUI and 'figure out' where to make the changes.

And so it really does take a decade plus to build a deep understanding of how this system works. I've met people who've done it, its as impressive as it is tragic. Specific system configuration sits squarely on the *know* side of the fence, and the ease of access to that information needs to be highly optimized for. What's is on the other side of the fence is *understanding* how different system architectures and configurations are better for certain use cases, and the emerging properties they will have when deployed at scale. That takes seeing and experiencing many different configurations before that understanding will ever be built up. Unfortunately, most members of my organization have spent their entire career with this system, and therefore struggle to even imagine any alternatives.

I started my [learning collection](/learning/) at the end of last year, to collect all information I wanted to wanted, and be able to go back too, so I could understand the complex topic of software. I now have a folder for software engineering, leadership, languages, writing, music, and life. Only recently have I realized the true power in that has been how it optimized the speed of knowing, so I could finally build deep understandings of complex topics.

---
### To Summarize:
- It is impossible to understand a complex topic from a single source, due to the bottlenecks of our communication mediums. So to understand complex topics, we need to be constantly learning and taking in as much input as possible to build up our internal neural networks and our mental models of the world. There's no way around that.
- We need to be conscious of what needs deep understanding to be effective at, and what should just be known. Then, we need to optimize for quick and easy access to what should be known.
---
author: Kevin Kelly
title: How learning Japanese has changed the way I think about programming languages.
date: 2024-04-29
description: How similar translating languages can feel to translating our human thoughts to the language of the machine.
tags:
  - software
  - language
  - programming-languages
  - software-quality
categories: 
series: 
aliases: 
draft: false
---
I was having a conversation with a friend the other day, who like many people who get paid to write code, has no formal education in programming, like a CS degree or a bootcamp. They were talking about how their employer wanted them to learn C# (they had mostly been using python) for a potential .NET app, and so they had paid for a C# course.

While this makes sense as one of the quickest ways to get up to speed with a certain language, I recommended they pay attention to and learn the fundamental constructs that C#  shares with Python, and where it differs.

Most professional programmers and software engineers have used, or at least dabbled with, many different languages throughout their careers. To the point where switching to another language becomes almost trivial. We can design systems with these fundamental constructs, then implement them in pretty much any language.

Once we obtain that deep understanding of fundamental constructs, we can become very confident and proud of our ability to 'think in computer.' Which is essential, we are writing for computers after all. But we often fail to see the drawbacks in limiting our thinking like that. I knew that we should strive for clean, 'readable' code, but I didn't truly understand the essence of that goal until I tried to think in another spoken language.

---
### To understand or to be understood.
The Japanese word that inspired this line of thinking was 分かる (pronounced *wakaru*), which is a verb that *translates* to 'to understand.' I had learned it a few years back in college in preparation to study abroad, but covid had other plans for my time in college. Now as a graduate, I decided to start learning Japanese again because it was quite fun, and I'm no longer in a rush to learn it all within two years.

One of the many details I often brushed pass the first time around was whether verbs were *intransitive* or *transitive*. That whether or not the verb is performed by the subject on itself (intransitive) or on a direct object (transitive). Basically, in the sentence '*I* am running,' running is intransitive, because I, the subject, is what is running. On the other hand, in the sentence 'I am running the *dishwasher,*' running is transitive, because I, the subject, am performing the verb on the dishwasher, the direct object.

To my horror, I realized 分かる was intransitive. In my many conversations (mostly with myself, I have been terrified to make this exact kind of mistake online) I have had in Japanese these last few years, I have been using 分かる wrong, as a transitive verb.

The [Wanikani explanation](https://www.wanikani.com/vocabulary/%E5%88%86%E3%81%8B%E3%82%8B) of the word says it best, so I'll just quote it here:

> This word means to *understand*, which might surprise you if you remember that the kanji 分 means *part*. Think of it this way: to understand something you have to break it down into its parts to see how it works.
> 
> Watch out, though, because whereas in English you understand something, in Japanese the thing you understand is the grammatical subject (it's doing the action). So, a more literal meaning for this is *to be understandable*. In other words, there's no object, and 分かる is intransitive, meaning it happens on its own. The かる ending is a hint, because verbs with かる as okurigana (kana added to the kanji) are usually intransitive.

To me, this is a completely different *mental model*, almost a different way of thinking. Lets borrow that word *model* and take a look at at what I mean. If I wanted to say 'I understand that book' in Japanese, I would say「その本が分かる.」Lets break that down real quick:

| 'Word' | Pronounciation | Meaning                                    |
| ------ | -------------- | ------------------------------------------ |
| その     | sono           | that (object)                              |
| 本      | hon            | book                                       |
| が      | ga             | subject marker, marking その本 as the subject |
| 分かる    | wakaru         | to be understandable                       |

So then, lets take a look at how the models differ in each language:

| Term          | *Model*                | In a sentance          | Translation of the *model*        |
| ------------- | ---------------------- | ---------------------- | --------------------------------- |
| To understand | To understand          | I understand that book | その本を分かる (grammatically incorrect) |
| 分かる           | *To be understandable* | その本が分かる                | That book is understandable.      |

The only difference between the correct sentence and the one I would've said is that I would have replaced が, the 'subject marker,' with を, the 'direct object marker.' Instead of the book, I as the speaker would implicitly be the subject of that sentence. Which, of course, is a grammatically incorrect use of the Japanese verb, but makes perfect sense to me if I am using my English model. 

Now, whenever I do my Wanikani flashcards, I always try and take care to answer with the meaning that's the closest *model* to the word, which is not necessarily the closest *translation.*  

This fundamental difference in the modeling everyday situations is reflected even stronger in the closest translations of the words 'to like' and 'to want,' which are 好き (pronounced *suki*) and 欲しい (pronounced *hoshii*) respectively. You might be thinking, "Are they just like 分かる in the previous example, where they're just intransitive verbs?" Nope, they're ***adjectives***. 

| Term    | *Model*     | In a sentance    | Translation of the *model*               |
| ------- | ----------- | ---------------- | ---------------------------------------- |
| To like | To like     | I like that book | その本を好きだ (*very* grammatically incorrect) |
| 好き      | is likeable | その本が好きだ          | That book is likeable.                   |

I absolutely could not wrap my head around this general model when I first learned it. When we use adjectives in English, we are generally either saying an objective fact, or we preface our subjective opinions with the fact that they are, well, opinions. I mean, if you were struggling to grasp some LeetCode problem and asked me for help, and I walked over and said 'That problem is understandable.' You might think I'm insulting you!

But Japanese is a *heavily* context based language. So when I say a thing is understandable, it is generally understood to be my point of view of the situation, not a comment on whether the thing is easy to understand. 

--- 
### But what does this have to do with programming languages?
I think the weight of the word language in programming *language* is often lost on us. Just like how I have to rewire my brain to think in a Japanese model, I had to do the same when I was first learning to program, just in a model of computation. Thinking in variables, if/else blocks, loops, etc. But unlike when I use my English model in a Japanese sentence, using an unclear or incorrect model to solve a problem might not just result in harmless little grammar mistakes, but it could have real world consequences. Even if you're confident in you and your team's ability to 'think in computer,' you should have no faith in their ability to '*communicate* in computer.' Similar to Japanese, programming languages themselves can be dangerously context dependent, and far too much can get lost in translation.

Think back to the explanation of those Japanese sentences. Presuming you're a native English speaker, to understand the meaning of the Japanese sentences, you needed to know both:
- How the sentence was built via the *Japanese model.* 
- The message being communicated via the *English translation.*

I think the same thing is going on whenever we write programs. We build our code 'sentences' in a *computer model* to be executed. But the computer isn't the only one who needs to understand our code: people need to as well. 

So, we need to focus on being explicit in the intention of our code by including translation, the actual meaning we're trying to communicate, *within* the code.

Remember the mistranslation that can occur from earlier if we tried to communicate with a different model:
- "That problem is understandable."
- *(Is he insulting me?)* "I think its pretty difficult."

These same mistranslations, inaccurate assumptions about another's intention, will occur when we try and communicate in computer models. A dead simple example (imagining no documentation):
```swift
sleep(2) // minutes, seconds, microseconds? you might not have the context, but the computer knows.

let SECONDS = 1
sleep(2*SECONDS) // this is the exact same model with an included translation
```

A little anecdote I like to remember this concept by, when I went to sports camps as a kid, something we were always told was to "stay on the balls of your feet." I mistranslated their message of "keep your weight forward" as "keep those heels *off* the ground buddy." I have *massive* calves now. What may seem obvious to you might fly right over someone else's head.

But whats far more likely then a simple mistranslation, and what I'd wager you've encountered many more times, is when there is *nothing* to translate. When your reading some code and you sit there, thinking:

> What on *earth* were they trying to do here? What could their intention possibly have been?

At this point, all bets are off, who knows if you're going to translate that code correctly.

I could take the time to come up with some more examples, but I think I would have a hard time not accidentally plagiarizing these two excellent [CodeAesthetic](/blog/learning/code_aesthetic) videos. In the examples they give, pay attention to how the function (the model) of the code doesn't change at all, but the added context (the translation) helps make the code more understandable and maintainable.
- [Naming Things in Code (7:24)](https://www.youtube.com/watch?v=-J3wNP6u5YU)
- [Don't Write Comments (5:54)](https://www.youtube.com/watch?v=Bf7vDBBOBUA)

To take this one last step forward: when we fail to write our translations down, what we end up writing might not even match what we had in our heads. While thought is [deep and intricate](/blog/blog/knowing_vs_understanding), its also a by nature a [lossy medium](/blog/learning/language_is_a_bottleneck). Don't add more loss for no reason.

> "Writing the decisions down is essential. Only when one writes do the gaps appear and the inconsistencies protrude. The act of setting turns out to require hundreds of mini decisions, and if the existence of these that distinguishes clear, exact policies from the fuzzy ones."
> 
> --- Fred Brooks, [*The Mythical Man-Month*](/blog/learning/the_mythical_man_month)

If writing in the model of our native tongues requires "hundreds of mini decisions", imagine what happens when we try to skip directly to a foreign computational model without first distinguishing "clear, exact policies from the fuzzy ones."

---
### To Summarize:
- We are fundamentally translating from a human language mental model to a computational, digital model when we write software.
- We should always be as explicit as possible with our intentions when we write software, making clear our *translations* of the *models*, being very specific so we can:
	- Better communicate our intentions to our teams (and our future selves).
	- Confirm what we write matches our original intention.
	- Encapsulate those models into simple components we can easily think about the context of a system. 
	

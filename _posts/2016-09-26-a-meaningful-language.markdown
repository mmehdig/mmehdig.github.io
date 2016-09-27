---
layout: post
title:  A meaningful language
date:   2016-09-26 23:59:59 +0200
categories: blog
comments: true
---

> "the meaning of a word is its use in the language" -- Ludwig Wittgenstein [^PI43]

I am in Barcelona! [How come?] My plan was to skip the summer vacation and instead visit this city, people, and friends here. After a strange summer (long story), I am really happy that I had this trip planned before the summer. My dear human/smart-enough-robot reader! You must be wondering what connects this trip to the "meaning", "language", and Wittgenstein. The answer is simple: I have to update my individual study plan (ISP), a document in which I have to renew every year. I thought why not write something here. Thinking about this idea, the landlady ringed the door and we had an interesting communication (without any sophisticate technologies) in Catalan (which I know nothing). You can argue that I understood nothing! But at least I can point out that she wanted me to close windows because birds might come inside. Anyway, it's a good topic to start my blog.

The philosophical arguments about "understanding" and "meaning" is not a new thing. Let's skip all these history and jump to our today's topic: artificial intelligence. [Wow wait! What do you mean by skipping the history! You just mentioned Wittgenstein!] Ok, Wittgenstein had this great skeptical view which rejects dogmatic representations as a conception of meaning. Everything about a meaningful word is in its usage. This anti-systematic philosophy of Wittgenstein won't be helpful unless we observe its practical implications. For example this view (the usage as meaning) is very much compatible with using [Language Models](https://en.wikipedia.org/wiki/Language_model) for natural language processing tasks such as machine translation.

These statistical language models are based on observed language usage. Typically, they can be used as a generator which predicts the word based on the given context:

$${\displaystyle P(w_{t}|\mathrm {context} )\,\forall t\in V}$$

The recent efficient models which use low dimensional vector space representations became very popular in several NLP tasks. Despite all these new advancements, not all these sentence/phrase/word/character representations can model word usage as they promise. I would like to argue that a language model solely based on textual context not even misses a lot of aspects of situated context, it also misses my early argument from Wittgenstein: these are another dogmatic representations if we read them out of context. This problem was identified by Harnad (1990) as "Symbol Grounding Problem". The textual language models are similar to what Harnad calls "symbolic representation" which is useful for manipulation of symbols but in order to become a meaningful language it needs to be grounded in perception and action.

I like this lecture by Raymond J. Mooney (2013): "[Grounded Language Learning](http://videolectures.net/aaai2013_mooney_language_learning/)" which provides a nice introduction and motivation for incorporating language modeling with perception and action. During my first yeah as PhD student, I spent most of these times reading relevant material in computer vision, language modeling and different ways to represent meaning. Now I came back to my original motivations on on early days of my master thesis (two years ago), but with more insight on what I really want to do: Learn to compose the grounded symbols. Harnad (1990) example:

$$ Zebra = Horse\ \&\ Stripes $$

I will write about the idea in future.

Oh, by the way! what happened next with the landlady! She came back in an hour and gave me this book!

![english-catalan-book](/images/english-catalan.jpeg)


#### Footnotes

[^PI43]: #43 Philosophical Investigation, Ludwig Wittgenstein (1953).

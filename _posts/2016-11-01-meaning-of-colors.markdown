---
layout: post
title:  Meaning of colors
date:   2016-11-01 23:59:59 +0200
categories: blog
comments: true
---

We recently had [Matthew Stone](http://clasp.gu.se/news-events/e/?eventId=7248779128) as invited speaker in [our group](http://clasp.gu.se/). They have very [interesting work](http://mcmahan.io/lux/) with Brian McMahan on modeling grounded color semantics.

Let me give you a simple definition for grounded semantics, there are two ways to look at it:

1. Mapping between symbols in language (words, phrases, and etc) to perceptual representations. (color words and color representations such as RGB)
2. Finding common ground meaning for words in dialog. (negotiation of word meanings on the fly)

They are trying to model vagueness for color representations and give it possibility of being used in pragmatic reasoning (the examples are in demo section of their published code).

There are two recent work [a short paper](https://arxiv.org/abs/1606.03821) and [a poster](https://arxiv.org/abs/1609.08777) showing up in [EMNLP 2016](http://www.emnlp2016.net/program/program.html), which they are doing things that I am interested in. They basically exposed the idea of using encoder-decoder model for colors. It's like machine translation for colors. (translate the given color code to English, or with a given word what would be the most likely color). In recent machine learning seminar in Chalmers, I talked about these works, here is [the slides](https://docs.google.com/presentation/d/1esqSjNuwp3fmjjdOIZl8blUkB6-wdSNKqXzjd6YhZMQ) (I wasn't aware of the word-to-color work at that time).

What is my problem now?

Well, I am trying to make sense of many problems that I see if we consider [principles of compositionality](http://plato.stanford.edu/entries/compositionality/) for color semantics. For starter, I think negation is the most problematic part (what does "not red" mean?, can you incrementally learn compositional "not" in "not red"?). I will come with some code and examples later, this might be interesting for [representation learning](https://en.wikipedia.org/wiki/Feature_learning) community.

I am not going to publish any post next week. But hopefully the week after that, I will have some codes and explanation of what I am working on :D.

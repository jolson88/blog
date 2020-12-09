---
layout: post
title: "Monolith is not a four-letter word"
date: YYYY-MM-DD
categories: Programming
---

TODO (IDEAS: https://caseymuratori.com/blog_0015)

Somewhere along the lines in the last 20-30 years of programming, "Short" started to become used interchangeably with "Simple." And instead of optimizing for simplicity, developers started optimizing for shortness. It started in our inner loops and spiraled outwards in scope. Small functions led to small modules. Small modules led to small applications. Small applications led to small systems. Lines of code within a given context was the metric to measure against.

From now on, I shall start referring to this tendency as "Premature Simplification."

Monoliths were out, Microservices were in. "Monolith" was a four letter word. Large was Lame. Small was Smart. But I think many programmers (including myself) failed to see the pain we were bringing down upon ourselves with this push. Just because things are small does not mean they are simple.

TODO: how more pieces means more communication and how more pieces makes logic more wisely distributed and harder to follow.

TODO: Add how more pieces also means more mental concepts we have to retain and juggle when understanding and working with the system.

A single 200+ line function can indeed be easier to understand and work with compared to 20 functions with 10+ lines.

POSSIBLY: you might think "but this makes it easier to reuse functions!" That's true. But will that make the code better? Or will it make it much easier to try to overly reuse the code and end up introducing a spaghetti monster of code with many more dependencies that have to be managed and make it more difficult for us to modify the code over time to meet our changing needs?

"Makes things as small as possible but no smaller."

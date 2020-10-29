---
layout: post
title: "FP and OOP are not enemies"
date: 2020-11-05
categories: Programming
---

This topic has been at the back of my mind since I saw people complaining about the addition of classes to JavaScript. Some people felt JavaScript was better suited as a Functional Programming language and providing better support for Object-Oriented Programming would lead to diluted and bad JavaScript code and coding practices.

I'm not going to discuss whether classes belong in JavaScript. I want to take a step back and ask a simple question: Are Functional Programming and Object-Oriented Programmer really all that different at their core? Is the energy expended debating between FP and OOP well spent?

**Spoiler Alert**: No, I don't think it's energy well spent. I don't believe that FP and OOP are enemies. And I think a lot of people miss the point of the both of them solving the same underlying problem. But to unwind that, let's go back to basics.

# What is a computation?

TODO (the nature of a computation)

TODO (the difference engine and analytical engine)

TODO (The First Programmer: Ada Lovelace)

# Where did computers come from?

TODO (The undecidability problem and the work of Turing and Alonzo Church)

# What is computer programming?

TODO (Data Structures and Algorithms)

- In other words: Information, and Operators that manipulate Information
  - Caveat: As shown in the Hardware Architecture / Von Neumann model of computing, Operators can be stored as Information itself. So it's really just all Information, but I won't be going into that meta-topic in this post.

TODO (a computer programming language therefore is...)

# Lineage of FP and OOP

TODO

TODO (create or find a lineage "graph"; possibly include Smalltalk to show branch between Simula->C++ lineage and Smalltalk/Objective-C; do we also include CSPs and Erlang?)

# The need for abstraction and modules

TODO (Barbara Liskov stuff; relationship between FP and modules compared to Objects (and possibly namespaces))

- What's the difference between a Module and an Object?
  - You might just as easily be able to say they are practically identical from the abstract sense of Information and Operators

# Managing state

TODO (what is state?)

TODO (managing state)

- But Jason, Objects **mutate** their state, don't they?
  - Some languages may have their languages do that, but it doesn't have to be that way.
  - Example: Erlang/OTP and FSM pattern (where state is immutable and the operation returns a new version of the state that should be managed by the system)
  - React developers may even start to sense some similarities here on how React manages state through the use of Context (Information) and Reducers (Operators).

# Is one mental model better than another?

TODO (Feynman on knowledge vs. understanding discussing two different models A and B that both ultimately share the same "consequences")

Reference Feynman’s knowledge versus understanding video (models A and B that both reach the same conclusions). They are mathematically equivalent. But the models can help us look at problems with different lenses. It’s common to know many different models that you apply depending on the problem (light as a wave vs light as a particle). What might involve a small change in one model may require a large effort in the other.

# So are FP and OOP competing against each other?

TODO (no, they are two different but complementary models for us programmers to think about computation. They give us different ways to express what we want the computer to do. For some sets of problems, FP may ultimately be the best way to express it (TODO; one data structure and 100 algorithms)). For other problems, an OOP-based approach may provide a better model to reason about code.

So please, let's stop the Holy Wars that try to pit FP and OOP against each other (I've most definitely been guilty of this in the past myself). Let's realize instead that they both provide value in different ways and there's not a reason to avoid learning both models.

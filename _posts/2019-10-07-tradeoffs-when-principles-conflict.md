---
layout: post
title: "Finding tradeoffs when principles conflict"
date:   2019-10-07
categories: Programming
---

As I find myself home sick and exploring Rust features on [Compiler Explorer](https://godbolt.org/) in bed, I started thinking about the process of making tradeoffs when your core principles are in conflict. How do you decide which path to take when you may not be able to "have your cake and eat it too"? I think it is these types of decisions that make software design so challenging and interesting.

This challenge is much more universal than just Rust. It's practically a software truism that exists at the heart of software engineering itself. Just because you have to make a choice between two conflicting choices doesn't mean that your core design is wrong or that you made a mistake. Software is about give-and-takes, the tradeoffs you have to make during the course of implementation.

# An example with Rust
[Rust's website](https://www.rust-lang.org/) outlines three major principles that Rust aims for:
- "**Performance**. Rust is blazingly fast and memory-efficient: with no runtime or garbage collector, it can power performance-critical services, run on embedded devices, and easily integrate with other languages."
    - *NOTE*: You may find different core language-team members emphasizing the goal of "Zero-Cost Abstractions."
- "**Reliability**. Rust’s rich type system and ownership model guarantee memory-safety and thread-safety — and enable you to eliminate many classes of bugs at compile-time."
    - *NOTE*: Often you will hear this principle referred to as a focus on **Safety**.
- "**Productivity**. Rust has great documentation, a friendly compiler with useful error messages, and top-notch tooling — an integrated package manager and build tool, smart multi-editor support with auto-completion and type inspections, an auto-formatter, and more."

These are big reasons that make Rust such a powerful systems programming language. And because of its focus on productivity, I agree with many others that it makes systems programming much more approachable to a larger number of programmers. While C++ is powerful too, I think it would be a stretch by any imagination to refer to C++ as "simple to learn."

Can Rust achieve all three at the same time though? I think there are times where you can find the goal of safety interfering with striving for performance. Implementing runtime checks for buffer overflows, integer overflows, integer underflows, division-by-zero, etc. certainly make applications more safe. But that's also a lot of code that could interfere with top-notch performance (especially if within a core loop and involving extra function invocations or memory allocations)

Interestingly, even the most basic of Rust code is immediately presented with these potential conflicts.

# When safety and performance potentially conflict
Safety nearly always comes at a cost. When we are writing safe code, we may care very deeply about bounds checking, integer overflows, or null pointer checks. Many gnarly bugs in software today come from mistakes arising from these. But we also need to realize that these checks come at a cost. To perform the checks means more code is potentially executing at run-time performing the checks, and that ultimately leads to slower code by definition. (The fastest code ever is the code that never runs in the first place)

Let's take a look at a trivial function that adds two numbers together in the C programming language:

```c
int add(int x, int y) {
    return x + y;
}
```

It is readily apparent in the [compiled assembly](https://godbolt.org/z/5HM-9h) that no safety checks, like integer overflow, are occurring:

```asm
add:
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        mov     DWORD PTR [rbp-8], esi
        mov     edx, DWORD PTR [rbp-4]
        mov     eax, DWORD PTR [rbp-8]
        add     eax, edx
        pop     rbp
        ret
```

Now let's take a peek at the Rust equivalent:

```rust
pub fn add(x: i32, y: i32) -> i32 {
    return x + y
}
```

Rust (minus optimizations) adds some checks for us:

```asm
example::add:
        push    rax
        add     edi, esi
        seto    al
        test    al, 1
        mov     dword ptr [rsp + 4], edi
        jne     .LBB0_2
        mov     eax, dword ptr [rsp + 4]
        pop     rcx
        ret
.LBB0_2:
        lea     rdi, [rip + .L__unnamed_1]
        mov     rax, qword ptr [rip + core::panicking::panic@GOTPCREL]
        call    rax
        ud2

str.0:
        .ascii  "./example.rs"

str.1:
        .ascii  "attempt to add with overflow"

.L__unnamed_1:
        .quad   str.1
        .quad   28
        .quad   str.0
        .quad   12
        .long   2
        .long   12
```

Immediately, we are checking for integer overflow. And [less trivial examples](https://godbolt.org/z/bHrN09) may add even more checks:

```rust
pub fn sum(xs: &[i32]) -> i32 {
    let mut sum = 0;
    for i in 0..xs.len() {
        sum += xs[i];
    }
    return sum
}
```

In this new example, Rust is checking for many different safety issues and, with some of them, it's not immediately clear why the checks are needed (hint: some of them are from smart wrappers around pointer arithmetic):
- Attempt to add with overflow
- Potentially calling `unwrap()` on a `None` value
- Attempt to calculate remainder with a divisor of zero
- Attempt to copy to overlapping memory
- Attempt to copy to unaligned or null pointer
- Attempt to copy from unaligned or null pointer

In these specific examples, Rust has a compromise though: optimizations. If you compile your code with optimizations on (`-O`), many of these issues [go away](https://godbolt.org/z/vL5a9k) (down from 1036 lines of assembly code to 83).

These tradeoffs are most definitely not specific to Rust. We face these types of conflicts all the time in software development:
- Speed vs. Space in algorithms (perhaps we consume more memory to have faster performing algorithms, or we have slower algorithms because we have less memory we can consume)
- Debugging vs. Performance (the cost and overhead of instrumenting code vs. the insight into the code we have if something goes wrong)
- Parity between software running in production and local development vs. our desire for quick and simple local development experiences and testing turnaround.

Even the real world is full of compromises, conflicts and trade-offs. We may feel a conflict between our desire for privacy vs. wanting to be kept safe from terrorists. We may feel a conflict between our desire for a better home or car vs. our desire to save for retirement. We may feel a conflict between our desire to support local businesses vs. the convenience of larger internet businesses like Amazon.

So how do we solve these apparent conflicts? I don't have answers here. As is often true in software, I think the answer is "it depends." But over the years I've come across several strategies that different folks believe in and use.

# Priority of Principles
I'm reminded of an old adage in the real-estate industry:

> "Price, Quality, Location: Pick Two"

We may feel we care about all our principles equally, but that may not be the case. After thinking deeply, we may realize our principles are actually prioritized. We may be more than willing to sacrifice a lower priority principle in order to preserve a higher priority one.

But sometimes, we may not be able to make this decision. Perhaps it feels a bit like [Sophie's Choice](https://www.urbandictionary.com/define.php?term=Sophie%27s%20choice); both choices are unbearable and a decision may put you in a no-win situation. In this case, we need to think about our situation differently or see if there is a way we can reframe our conflict to find a better winner.

Or, to quote the movie [Wargames](https://www.youtube.com/watch?v=6DGNZnfKYnU): "The only winning move is not to play." It may be a sign that what we are trying to achieve that led us to this tradeoff is not in our best interest as we originally thought. Sometimes the hardest decision in software is deciding what **not** to do.

# Principle of Least Surprise
The gist of the Principle of Least Surprise, also known as the [Principle of Least Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment), is that a given result should not be astonishing or surprising to an end-user. In other words, a feature should behave the way that users expect it to behave.

For example, Rust often touts its goal of developing safe software. If a Rust developer was learning how to do pattern matching and found out that by using it, the application could easily suffer a null-pointer exception or buffer overflow, that behavior would be highly surprising to the developer. This would not be a good experience and would lead one to believe that the feature was very poorly designed.

This exists in user interface design as well. On PCs, it is standard for the F1 key to bring up Help files. If the application takes over the F1 key and has it do some other action instead, that can be highly surprising. Or, how about Ctrl+C and Ctrl+V for copy and paste on Windows. It would be highly surprising (and probably frustrating) if a text editor you were using overrode those key sequences to mean something else entirely.

So when we are faced with a decision whose outcome may violate one or more of our principles, we could ask ourselves which behavior would result in the least surprise to our users.

# Falling into the Pit of Success
> The Pit of Success: in stark contrast to a summit, a peak, or a journey across a desert to find victory through many trials and surprises, we want our customers to simply fall into winning practices by using our platform and frameworks. To the extent that we make it easy to get into trouble we fail.
> - Rico Mariani, Microsoft

Falling into the [Pit of Success](https://blog.codinghorror.com/falling-into-the-pit-of-success/). While Rico Mariani talked about this in the context of programming language design, I think it applies much more widely than that. You may have heard other similar sayings as well: "Make it easy to do the right thing, and make it hard to do the wrong thing", "Good By Default", etc.

Things don't usually become hard to use because of a consequence of a single decision in isolation. Complexity can often come about from a form of "death by a thousand cuts." Decision after decision compound until you are left with something that is difficult to comprehend.

> I often think of C++ as my own personal Pit of Despair Programming Language. Unmanaged C++ makes it so easy to fall into traps. Think buffer overruns, memory leaks, double frees, mismatch between allocator and deallocator, using freed memory, umpteen dozen ways to trash the stack or heap – and those are just some of the memory issues. There are lots more "gotchas" in C++. C++ often throws you into the Pit of Despair and you have to climb your way up the Hill of Quality. (Not to be confused with scaling the Cliffs of Insanity. That's different.)
> - Eric Lippert

So if you are stuck trying to make a decision between two different options that are in conflict, it is often still useful to ask yourself whether the decisions you are making allow your users to do the better thing by default. Will you be encouraging them to "fall into the pit of success"?

# The Art of Simplicity
> A primary cause of complexity is that software vendors uncritically adopt almost any feature that users want.
> - Niklaus Wirth

The longer I write software, the more I find myself striving for simplicity. Finding an elegant and simple solution to a problem is perhaps the hardest thing to achieve as well, in my opinion. Complexity is easy. So rarely do we ask ourselves whether a feature actually adds enough value to justify its expense. Does the cost of a feature (both from a programmer's time and the added complexity to a user's understanding of our software) justify its own existence?

> Perfection is achieved, not when there is nothing more to add, but when there is nothing left to take away.
> - Antoine de Saint-Exupery

It seems that we so often build software that strives to solve every corner case and address every problem in the domain. But with each successive feature we add, we are also making our software slower, harder to maintain, and harder to use. Every new feature added to a compiler is likely to make the compiler take longer to run. Every new feature added to a web service requires the developers to keep more concepts in their mind when trying to find bugs or develop future features.

Not only is it hard to refine a problem down to just its core essence ("essential complexity" as Brooks put it and is discussed in the [Ball of Mud](http://laputan.org/mud/) paper), but you also have to be careful to not oversimplify it either.

> Everything should be made as simple as possible, but no simpler
> - Albert Einstein

Sadly, it seems we are seldom given the time necessary to learn from the work we are doing and properly refine our software. In the meantime, our software is getting slower more quickly than hardware is getting faster. Our operating systems and applications feel more bloated than they ever have been.

# Others?
Have you heard of other approaches to dealing with tradeoffs? When you are faced with hard decisions around making a tradeoff in your design, how do you approach it?

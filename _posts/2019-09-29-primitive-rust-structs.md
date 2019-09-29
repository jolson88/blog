---
layout: post
title:  "Primitive Rust: a dive into structures"
date:   2019-09-29
categories: Programming
---

I've always loved learning about how computer stuff works. Even though I started coding late in life compared to my friends (college for me vs. when they were kids), I've also always felt pulled in an opposite direction. I started with Classic ASP sites in VBScript, and shortly moved to C# programming. I'm a self-taught developer. And along the way, I was learning about all sorts of things that other people didn't seem to care much about.

I read Tannenbaum's "Operating System Design and Implementation" for fun. I became fascinated with Smalltalk and bought and read the Bluebook to dig into Smalltalk internals. I'm always fascinated by self-hosted programming languages even if the idea still stretches my brain. And many more projects along these lines. This desire has only grown over the years.

This desire of mine has become especially strong lately as software seems to somehow be getting slower and of lower quality even in the face of increasing capabilities given to us by hardware. Not only am I learning Rust because I love the language, but also because I'm wanting to write code at a lower level and to be more connected to the underlying hardware my code runs on.

I've always struggled putting this curiosity to work when learning a new topic though. I suck at trying to come up with project ideas out of the blue to spend time on. So as I'm learning Rust, I'm going to do "projects" that are peaking under the hood and understanding at a deeper level. I'm creating these posts to share this journey with others.

In this first post, I'm going to dive into "Primitive Rust" to take a peak at structures. I'm going to take it nice and easy at first and deal with more complex types later.

# Size of primitive types
The size of many primitive types are not surprising, as the number of bits is in the type itself. We can verify this using the `size_of` function in `std::mem`. These initial numbers I will get by using the [Rust playground](https://play.rust-lang.org/).

```rust
> std::mem::size_of::<u8>()
1

> std::mem::size_of::<u32>()
4

> std::mem::size_of::<i64>()
8

> std::mem::size_of::<bool>()
1
```

*Note: You can find more information in the [size_of documentation](https://doc.rust-lang.org/std/mem/fn.size_of.html).*

`usize` and `isize` are a bit different as their size depends on the target machine being compiled to. These two types are meant to be able to contain every possible memory address on the target platform. So at least 4 bytes for 32-bit systems, and 8 bytes for 64-bit systems.

# Structure sizes
Let's add a couple of these types to a structure to dig more into how structures are laid out. When we put primitive types into a structure, do we get the size we expect?

```rust
struct Foo {
    first: u32,
    second: u32
}

let x = std::mem::size_of::<Foo>();
// x = 8
```

A `u32` is 4 bytes, and we have two of them, so we get 8 bytes as expected. Let's try adding a `bool` to the end of our `Foo` structure.

```rust
struct Foo {
    first: u32,
    second: u32,
    third: bool
}

let x = std::mem::size_of::<Foo>();
// x = 12
```

If you are used to using higher-level languages like Java, C#, or JavaScript, this may be a surprise to you. We have two `u32` types taking up 8 bytes, and a `bool` taking up a single byte. So you may have expected the total size of the `Foo` struct to be 9, not 12. Why are there three extra bytes in our struct?

# Understanding data layout and alignment
Even though you may not realize it, the fundamental architecture of computers we use today is based on a computer architecture devised all the way back in 1945 by mathematician and physicist John von Neumann. Today we refer to this computer architecture as the [von Neumann architecture](https://en.wikipedia.org/wiki/Von_Neumann_architecture). What does that have to do with understanding data layout and alignment? Because this architecture has a direct impact on how computers of today read to and from memory.

![von Neumann Architecture]({{ "/images/von-neumann.png" | absolute_url }})

This architecture has evolved over the years to mean a computer that fetches data and instructions using the same mechanism, preventing them from being fetched at the same time (as opposed to the opposing Harvard architecture machine). The CPU and Memory Unit are connected by a bus. When we refer to the bit-ness of our CPU, we have historically been referring to the number of bits that can be sent over this bus. 32-bit CPUs have instructions that are 4 bytes long and can address a memory space of up to 32-bits (2^32, or about 4GB). 64-bit CPUs can reference a 64-bit memory space (2^64, or about 16 exabytes).

This bus also determines how computer hardware most efficiently transfers day to and from memory. On a 32-bit computer, the computer is most efficient when it is working with data in 4-byte increments that start on a 4-byte boundary. This is known as being naturally aligned. If data is not naturally aligned on these boundaries, the CPU may need to retrieve extra values from memory to work with a single piece of data. For example, if a 32-bit integer (4 bytes long) starts on the 3rd byte from a boundary, the CPU would need to fetch the first 4-bytes to get the 1st byte of the number and then fetch the next 4 bytes to get the last 3 bytes of the integer.

This is why many computer languages add padding to structures to have them more naturally align to the architecture of the target platform. In our example above, even though the underlying types within the structure only occupied 9 bytes, the size of our structure ended up being 12 bytes (on a 32-bit platform). This is because Rust's default struct representation automatically aligned our structure on natural boundaries.

# Quick Aside: The rise of CPU caches
As our CPUs have gotten faster and faster, the ability to quickly transfer data between the memory unit and the CPU has become more problematic. This is known as the [von Neumann bottleneck](https://en.wikipedia.org/wiki/Von_Neumann_architecture#Von_Neumann_bottleneck). While the von Neumann bottleneck is usually in reference to the fact that instructions and data share the same underlying bus, we are also facing a similar problem because of the speed of signal propagation within the hardware itself.

A common rule of thumb is that for a signal to travel around 15cm on a PCB, it takes roughly 1 nanosecond. In this regard, the distance between our main memory and the CPU on the motherboard is absolutely huge. A 3ghz processor has a clock cycle of roughly 0.3 nanoseconds. The time it takes to propagate just the electrical signal itself out to main memory and back is 2 nanoseconds. That's the fastest part of the trip too!

So the signal propagation alone has our CPU unable to do the work it needs to on that data for at least 6 cycles. Even the fastest external memory we have today runs at a much slower clock rate than our CPUs though. It's not uncommon for a fetch to external memory to take as many as 200+ cycles on a modern desktop computer. When writing high-performance software, that number might as well be eternity. And if memory seems that slow, you can only image how slow disk I/O is :P.

CPU makers have their CPUs do all sorts of fancy things to help keep the CPU active while these fetches are happening. They do out-of-order and speculative execution, and they also bring memory much closer to the CPU (on the same dye as the CPU itself) in the form of caches. [CPU caches](https://en.wikipedia.org/wiki/CPU_cache) are an absolutely fantastic topic to read about. There are some [great](https://www.youtube.com/watch?v=rX0ItVEVjHc) [talks](https://www.youtube.com/watch?v=WDIkqP4JbkE) that discuss the importance of being aware of CPU caches. The game industry has been particularly passionate about this topic and have spawned a movement of sorts called Data-Oriented Design that grew out of this realization of how modern CPU architecture works today. 

# Structs at runtime
Let's take a look at how the examples we saw above match to what a simple program would be doing at runtime. We'll create a very simple main function that uses two instances of our first struct. We can see what the program will do by examining the assembly output of the program. Why assembly? Well, I pick that because it's as close as we can get to the hardware :).

```rust
struct Foo {
    first: u32,
    second: u32,
}

pub fn main() {
    let f1 = Foo {
        first: 1,
        second: 2,
    };
    let f2 = Foo {
        first: 3,
        second: 4,
    };
}
```

```asm
example::main:
        sub     rsp, 16
        mov     dword ptr [rsp], 1
        mov     dword ptr [rsp + 4], 2
        mov     dword ptr [rsp + 8], 3
        mov     dword ptr [rsp + 12], 4
        add     rsp, 16
        ret
```

Each field is exactly 4 bytes after each other when we are pushing onto the stack pointer (`rsp`). We can also see that the second instance of our Foo struct immediately follows the first instance and continues our 4 byte stride. That's what we expect given what we found above. But now let's verify what is happening when we add a `bool` value.

```rust
struct Foo {
    first: u32,
    second: u32,
    third: bool,
}

pub fn main() {
    let f1 = Foo {
        first: 1,
        second: 2,
        third: true,
    };
    let f2 = Foo {
        first: 3,
        second: 4,
        third: false,
    };
}
```

```asm
example::main:
        sub     rsp, 32
        mov     dword ptr [rsp], 1
        mov     dword ptr [rsp + 4], 2
        mov     byte ptr [rsp + 8], 1
        mov     dword ptr [rsp + 16], 3
        mov     dword ptr [rsp + 20], 4
        mov     byte ptr [rsp + 24], 0
        add     rsp, 32
        ret
```

As we noticed with size padding above, we see it in action here. Our second struct instance starts at an offset of 16 bytes when we are writing it to the stack, making it naturally aligned for our 64-bit CPU. So even though the contents of the struct technically only take up 9 bytes, we are operating on it in naturally aligned steps.

**NOTE**: *Depending on whether you are targeting a 32-bit or 64-bit architecture, you may find that the stack pointer offset for the start of the second Foo instance may be different than what I show here. I'm targeting a 64-bit architecture so the compiler has chosen a natural alignment of 8 bytes (64-bit architecture), hence the offset of 16 bytes. If you are targeting a 32-bit architecture (4 bytes), you might find that the second Foo instance is offset by 12 bytes instead.*

# Beware depending on runtime layout of structs
It is important to be cautious if you need to rely on the specific layout of a structure in memory at runtime. As quoted from the official [Rust docs](https://doc.rust-lang.org/nightly/reference/type-layout.html#the-default-representation):

> There are no guarantees of data layout made by this representation.

Why is this? Let's slightly alter the example above by changing the first field from a `u32` to a `u16`:

```rust
struct Foo {
    first: u16,
    second: u32,
    third: bool
}

pub fn main() {
    let f1 = Foo {
        first: 1,
        second: 2,
        third: true,
    };
    let f2 = Foo {
        first: 3,
        second: 4,
        third: false,
    };
}
```

If we then take a look at the [asm output](https://godbolt.org/z/HXUjy-) for this change, we'll see an interesting consequence of the change:

```asm
example::main:
        sub     rsp, 16
        mov     word ptr [rsp + 4], 1
        mov     dword ptr [rsp], 2
        mov     byte ptr [rsp + 6], 1
        mov     word ptr [rsp + 12], 3
        mov     dword ptr [rsp + 8], 4
        mov     byte ptr [rsp + 14], 0
        add     rsp, 16
        ret
```

The compiler has decided for us that at runtime, the second field should actually come first in memory. We can see this with the third instruction above:

```asm
mov     dword ptr [rsp], 2
```

There is no offset from the stack pointer (`rsp`). So our value `2` we are putting in the `second` field of our struct is being set as the first field in memory. Then, the first field that we changed to `u16`, and finally our `bool` value.

You see, the compiler is pretty smart and will do some of these optimizations for us if it deems it would be "In Our Best Interest." This is not how it works in all languages. Some languages make guarantees that they won't reorder structure fields at runtime. As we see above, Rust is not currently one of those languages and the docs explicitly tell us that the default representation of structures doesn't guarantee data layout.

If we need to preserve the order of fields, what can we do about it?

# Changing representation of structures
We may be working with a file format that with exact layout requirements. We may be trying to interoperate with a different programming language. Or we may be working with a network protocol with its own layout requirements. How can we have better control over the layout of our structures? Rust has provided us with the [`repr` attribute](https://doc.rust-lang.org/reference/type-layout.html#representations) for exactly this purpose.

One of the most common representations you are likely to come across is the [C representation](https://doc.rust-lang.org/reference/type-layout.html#the-c-representation). The obvious use of the C representation is when creating structures that we want to use to interoperate with C/C++ code. That is not the only usage though. By using the C representation, we are given a representation that is predictable and that we can rely on to do things like reinterpreting it as a different type or working with it at a primitive level with pointers.

We use the C representation by decorating our struct with the attribute `#[repr(C)]`:

```rust
#[repr(C)]
struct Foo {
    first: u16,
    second: u32,
    third: bool
}

pub fn main() {
    let f1 = Foo {
        first: 1,
        second: 2,
        third: true,
    };
    let f2 = Foo {
        first: 3,
        second: 4,
        third: false,
    };
}
```

```asm
example::main:
        sub     rsp, 16
        mov     word ptr [rsp], 1
        mov     dword ptr [rsp + 4], 2
        mov     byte ptr [rsp + 8], 1
        mov     word ptr [rsp + 16], 3
        mov     dword ptr [rsp + 20], 4
        mov     byte ptr [rsp + 24], 0
        add     rsp, 16
        ret
```

Where this same example above switched the order of the fields in the output, we can now see that the ordering is preserved. The `first`, `second`, and `third` fields are set in that exact order with stack pointer offsets. No longer is our `second` field being set first on the stack pointer.

Remember how we also noticed above that Rust aligned our structure on 4 or 8 byte boundaries by default? The `repr` attribute also allows us to control this through the use of `packed` and `align` values. `packed` allows us to use a smaller alignment than the default, and `align` allows us to use a larger one.

With the default representation, we saw a size of either 12 or 16 for the following struct (depending on 32-bit vs. 64-bit of our target architecture):

```rust
struct Foo {
    first: u32,
    second: u32,
    third: bool
}

let x = std::mem::size_of::<Foo>();
// x = 12
```

We can use `packed` to change to an alignment of 1 byte if we wish. We should remember that this could have negative consequences on the performance of your code since the CPU will potentially need to work harder to work with the data. Why would we want to do this if we know there are potential performance consequences? Well, we may need to conserve the memory if working in a memory-constrained system and we are fine with the memory/speed trade-off we are making. Or, we may be working with a disk format or a network protocol where the data alignment is critical to get correct.

```rust
#[repr(packed(1))]
struct Foo {
    first: u32,
    second: u32,
    third: bool
}

let x = std::mem::size_of::<Foo>();
// x = 9
```

As mentioned above, we can also use `align` to increase the alignment as well:

```rust
#[repr(align(8))]
struct Foo {
    first: u32,
    second: u32,
    third: bool
}

let x = std::mem::size_of::<Foo>();
// x = 16
```

# Wrap-up
So that was a quick look at plain ol' structures in Rust using primitive types and how they can be laid out in memory and used by the CPU at runtime. If you are new to this stuff, I hope you found any of this stuff useful. If you are more experienced with this stuff and want to correct any mistakes I made, feel free to leave a comment down below :). Until next time!


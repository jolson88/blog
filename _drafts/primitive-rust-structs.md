---
layout: post
title:  "Primitive Rust: Dive into structures"
date:   2019-09-30
categories: Programming
---

I've always loved learning about how computer stuff works. Even though I started coding late in life compared to my friends (college vs. when I was a kid), I've also always felt pulled in an opposite direction. I started with Classic ASP sites in VBScript, and shortly moved to C# programming. I'm a self-taught developer. And along the way, I was learning about all sorts of things that other people didn't seem to care much about.

I read Tannenbaum's "Operating System Design and Implementation" for fun. I became fascinated with Smalltalk and bought and read the Bluebook to dig into Smalltalk internals. I'm always fascinated by self-hosted programming languages even if the idea still stretches my brain. And many more projects along these lines. This desire has only grown over the years.

This desire of mine has become especially strong lately as software seems to somehow be getting slower and of lower quality even in the face of increasing capabilities given to us by hardware. Not only am I learning Rust because I love the language, but also because I'm wanting to write code at a lower level and to be more connected to the underlying hardware my code runs on.

I've always struggled putting this curiosity to work when learning a new topic though. I suck at trying to come up with project ideas out of the blue to spend time on. So as I'm learning Rust, I'm going to do "projects" that are peaking under the hood and understanding at a deeper level. I'm creating these posts to share this journey with others.

In this first post, I'm going to dive into "Primitive Rust" to take a peak at structures. I'm going to take it nice and easy at first and deal with more complex types later.

# Size of Primitive Types
The size of many primitive types are not surprising, as the number of bits is in the type itself. We can verify this using the `size_of` function in `std::mem`.

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

`usize` and `isize` are a bit different as their size depends on the target machine being compiled to. In the case of targetting 32-bit systems, `usize` will be 4 bytes long. On the other hand, `usize` will be 8 bytes in the case of 64-bit systems. This is because these two types are meant to be able to contain every possible memory address on the target platform. Hence, 4 bytes for 32-bit systems, and 8 bytes for 64-bit systems.

# Structures at Runtime
Let's add a couple of these types to a structure to dig more into structures at runtime. When we put primitive types into a structure, do we get the size we expect?

```rust
struct Foo {
    first: u32,
    second: u32
}

let x = std::mem::size_of::<Foo>();
// x = 8
```

A `u32` is 4 bytes, and we have two of them, so we get 8 bytes as expected. Let's try adding a bool to the end of our `Foo` structure.

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

# The impact of layout/alignment on structures
TODO (Discuss layout/alignment)

# Structs at runtime
TODO (show simple main program that builds the struct on the stack and increases the value of one of the fields; emphasize "zero-cost" as ptr and ptr offsets)

```rust
TODO
```

```asm
TODO
```

Each field is exactly 4 bytes after each other when we are pushing onto the stack pointer (`rsp`). That's what we expect given what we found above. But now let's verify what is happening when we add a `bool` value.

```rust
TODO
```

```asm
TODO
```

TODO (show simple output when a bool is added to struct and stack pointer walks by 4 bytes for each field)

# Beware depending on runtime layout of structs
It is important to be cautious though if you need to rely on the specific layout of a structure in memory at runtime. As quoted from the official [Rust docs](https://doc.rust-lang.org/nightly/reference/type-layout.html#the-default-representation):

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

You see, the compiler is pretty smart, and will do some of these optimiziations for us if it deems it would be "In Our Best Interest." So if you need to preserve the order of fields (let's say you are needing to pass this struct to some external C code), what can you do about it? Well, this is why Rust has provided us with the [`repr` attribute](https://doc.rust-lang.org/reference/type-layout.html#representations).

# Changing representation of structures
TODO (what if you need to keep the memory layout exact, like if you are calling out to native code in C; show repr(C))

Remember how we also noticed above that Rust aligned our structure on 4-byte boundaries by default? The `repr` attribute also allows us to control this through the use of `pack` and `align` values.

TODO (Show `alignment` and `pack`)

# Wrap-up
TODO

---
title: "Ownership in Rust"
excerpt: Explaining how ownership in Rust works.
permalink: /posts/2025/02/01/rust-ownership.md/
author: "Austin B."
---

# Preface

I am *not* an expert in Rust. In fact, I have just recently taken up the endeavor or learning Rust. I am currently finishing my first end-end read of the _The Rust Programming Language_ by Steve Klabnik and Carol Nichols. This book is **fantastic** and I highly recommend it for learning.

This page serves as a summary of what I learned from that book on _ownership_ in Rust as well as some other helpful resources for clarification and/or review.

Enjoy :crab:

---

# Resources

- [*The Rust Programming Language*](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)
- [The Rust Survival Guide](https://www.youtube.com/watch?v=usJDUSrcwqI&themeRefresh=1)

---

# Understanding Ownership

**Ownership** is Rust's most unique feature. It enables Rust to make memory safety guarantees without needing a garbage collector, so it's important to understand how ownership works.

## What is Ownership?

*Ownership* is a set of rules that govern how a Rust program manages memory. All programs have to manage the way they use a computer's memory while running. 

Some languages have garbage collection that regularly looks for no longer used memory as the program runs. In other languages, the programmer must explicitly allocate and free the memory.

Rust uses a third approach: memory is managed through a system of ownership with a set of rules that the compiler checks. If any of the rules are violated, the program won't compile. None of the features of ownership will slow down your program while it's running.

### Ownership Rules

First, let's take a look at the ownership rules. Keep these rules in mind as we work through the examples that illustrate them:

- Each value in Rust has an *owner*.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

### Variable Scope

A *scope* is the range within a program for which an item is *valid*. Take the following variable:

```rust
let s = "hello";
```

The variable `s` refers to a string literal, where the value of the string is hardcoded into the text of the program. The variable is valid from the point at which it's declared until the end of the current *scope*.

```rust
{                       // s is not valid here, it's not yet declared
    let s = "hello";    // s is valid from this point forward

    // do stuff with s
}
                        // this scope is now over, and s is no longer valid
```

In other words, there are two important points in time here:

- When `s` comes *into* scope, it is valid.
- It remains valid until it goes out of scope.

At this point, the relationship between scopes and when variables are valid is similar to that in other languages.

### The `String` Type

The `String` type manages data allocated on the heap and as such is able to store an amount of text that is unknown to us at compile time. You can create a `String` from a string literal using the `from` function, like so:

```rust
let s = String::from("hello");
```

The double colon `::` operator allows us to namespace this particular `from` function under the `String` type rather than using some sort of name like `string_from`.

This kind of string can be mutated:

```rust
let mut s = String::from("hello");
s.push_str(", world!");
println!("{s}");
```

### Memory and Allocation

With the `String` type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on the _heap_, unkown at compile time, to hold the contents. This means:

- The memory must be requested from the memory allocator at runtime.
- We need a way of returning this memory to the allocator when we're done with our `String`.

That first part is done by us: when we call `String::from`, its implementation requests the memory it needs. This is pretty much universal in programming languages.

In languages with a *garbage collector* (GC), the GC keeps track of and cleans up memory that isn't being used anymore, and we don't need to think about it.

In most languages without a GC, it's our responsibility to identify when memory is no longer being used and to call code to explicitly free it, just as we did to request it. Doing this correctly has historically been a difficult programming problem. If we forget, we'll waste memory. If we do it too early, we'll have an invalid variable (such as a _dangling reference_). If we do it twice, that's a bug too. We need to pair exactly one `allocate` with exactly one `free`.

Rust takes a different path: the memory is automatically returned once the variable that owns it goes out of scope. Here's a version of our scope example using a `String`:

```rust
{
    let s = String::from("hello");  // s is valid from this point forward

    // do stuff with s
}                                   // this scope is now over and s is no longer valid
```

There is a natural point at which we can return the memory our `String` needs to the allocator: when `s` goes out of scope. When a variable goes out of scope, Rust calls a special function for us. This function is called `drop`, and it's where the author of `String` can put the code to return the memory. Rust calls `drop` automatically at the closing curly bracket.

#### Variables and Data Interacting with Move

Earlier, we said that when a variable goes out of scope, Rust automatically calls the `drop` function and cleans up the heap memory for that variable. But if we have two variables with pointers to the same location in memory, this is a problem. If Rust were to free both, then we would get a *double free* error. Freeing memory twice can lead to memory corruption, which can potentially lead to security vulnerabilities.

Rust considers the previous variable bound to the pointer as invalid. Therefore, Rust doesn't need to free anything when the previous variable goes out of scope.

If you try to use an invalidated reference, Rust will prevent you.

If you've heard the term *shallow copy* and *deep copy* while working with other languages, the concept of copying the pointer, length, and capacity without dopying the data probably sounds like making a shallow copy. But because Rust also invalidates the first variable, instead of being called a shallow copy, it's known as a *move*.

#### Scope and Assignment

The inverse of this is true for the relationship between scoping, ownership, and memory being freed via `drop` function as well. When you assign a completely new value to an existing variable, Rust will call `drop` and free the original value's memory immediately.

#### Variables and Data Interacting with Clone

If we *do* want to deeply copy the heap data of the `String`, not just the stack data, we can use a common method called `clone`.

#### Stack-Only Data: Copy

Types of data such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual values are quick to make. That means there's no reason we would want to prevent such a variable from being valid after we create another variable assigned to it. In other words, there is no difference between deep and shallow copying here, so calling `clone` wouldn't do anything different from the usual shallow copying. 

Rust has a special annotation called the `Copy` trait that we can place on types that are stored on the stack, as integers are. If a type implements the `Copy` trait, variables that use it do not move, but rather are trivially copied, making them still valid after assignment to another variable.

Rust won't let us annotate a type with `Copy` if the type has implemented the `Drop` trait. If the type needs something special to happen when the value goes out of scope and we add the `Copy` annotation to that type, we'll get a compile-time error.

So, what types implement the `Copy` trait? As a general rule, any group of simple scalar values can implement `Copy`, and nothing that requires allocation or is some form of resource can implement `Copy`.

### Ownership and Functions

The mechanics of passing a value to a function are similar to those when assigning a value to a variable. Passing a variable to a function will move or copy, just as assignment does.

### Return Values and Scope

Returning values can also transfer ownership.

The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by drop unless ownership of the data has been moved to another variable.

While this works, taking ownership and then returning ownership with every function is a bit tedious. What if we want to let a function use a value but not take ownership? It’s quite annoying that anything we pass in also needs to be passed back if we want to use it again, in addition to any data resulting from the body of the function that we might want to return as well.

But this is too much ceremony and a lot of work for a concept that should be common. Luckily for us, Rust has a feature for using a value without transferring ownership, called references.

***

# Summary

Overall, **ownership** is Rust's way of making sure that before we compile our code, we are *sure* that pesky bugs in basic memory management won't occur. Memory management is a complex task and making your code memory safe in an unsafe language can be a difficult task. Ownership is really the first step Rust takes to ensure that our code is memory safe.


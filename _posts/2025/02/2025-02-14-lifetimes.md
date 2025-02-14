---
title: "Lifetimes in Rust"
excerpt: Explaining how lifetimes in Rust work.
permalink: /posts/2025/02/014/lifetimes.md/
author: "Austin B."
---

# Preface

I am *not* an expert in Rust. In fact, I have just recently taken up the endeavor or learning Rust. I am currently finishing my first end-end read of the _The Rust Programming Language_ by Steve Klabnik and Carol Nichols. This book is **fantastic** and I highly recommend it for learning.

This page serves as a summary and my own reference of what I learned about _generic lifetimes_ in Rust as well as some other helpful resources for clarification and/or review.

Enjoy :crab:

---

# Resources

- [*The Rust Programming Language*](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)

# Lifetimes

**Lifetimes** are how the compiler keeps track of how long references are valid. Every reference in Rust has a lifetime, even if it's not explicitly written.

Lifetimes ensure that a reference is never used after the value it points to has been dropped (gone out of scope). This prevents dangling references.

# Validating References with Lifetimes

Lifetimes are a kind of _generic_ type. But instead of ensuring that type has a certain kind of behavior, lifetimes ensure that references are valid as long as we need them to be.

**Every reference in Rust has a *lifetime***.

The **lifetime** is the scope for which that reference is valid.

Most of the time, lifetimes are implicit inferred, just like most of the time types are inferred. We must annotate types only when multiple types are possible. In a similar way, we must annotate lifetimes when the lifetimes of references could be related in a few different ways.

## Preventing Dangling References

The main aim of lifetimes is to prevent *dangling references*, which cause a program to reference data other than it's intended reference. For example, a use-after-free.

```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {r}");
}
```

The outer scope declares a variable named `r` with no initial value (this is allowed in Rust even though Rust does not allow null values). 

The inner scope declares a variable named `x` with the initial value of `5`. Inside the inner scope, we attempt to set the value of `r` as a reference to `x`.

After the inner scope ends, we attempt to print the value in `r`.

This code *won't* compile because the value `r` is referring to has gone _out of scope_ before we try to use it.

Here is the error message:

```
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
5 |         let x = 5;
  |             - binding `x` declared here
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 |
9 |     println!("r: {r}");
  |                  --- borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

We can see that the error message says that `x` "does not live long enough".

The reason is that `x` will be out of scope when the inner scope ends on line 7. But `r` is still valid for the outer scope; because its scope is longer we say that it "lives longer".

If Rust allowed this code to work, `r` would be referencing memory that was deallocated when `x` went out of scope, and anything we tried to do with `r` wouldn't work correctly.

Rust determines that this code is invalid by using a **borrow checker**.

See more on the borrow checker for a basic look at how it validates references through lifetimes (with a great example) [here](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#the-borrow-checker).

## Generic Lifetimes in Functions

Suppose we want to write a function that takes in two string slices (which are references) and returns the longest. Here is how we would use it:

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2.as_str());
    println!("The longest string is {result}");
}
```

Note: The `String` method `as_str()` returns the `String` as a `&str` type.

We want the function to take string slices rather than strings because we don't want the `longest` function to take ownership of its parameters. We want it to _borrow_ those values.

Consider the following implementation of `longest`:

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

If we try to compile this, we get the following error:

```
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

Notice the `help` text. This reveals that the return type of the function needs a generic lifetime parameter on it because Rust can't tell whether the reference being returned refers to `x` or `y`.

We don't know the concrete values that will be passed into this function, so we don't know whether the `if` case of the `else` case will execute.

We also don't know the concrete lifetimes of the references that will be passed in, so we can't look at the scopes to determine whether the reference we return will always be valid.

The borrow checker can't determine this either because it doesn't know how the lifetimes of `x` and `y` relate to the lifetime of the return value.

To fix this error, we'll add **generic lifetime parameters** that define the relationship between the references so the borrow checker can perform its analysis.

---

### Purpose of Generic Lifetimes: Detailed Explanation

Let's explain further. Consider this configuration of `main` calling the `longest` function:

```rust
fn main() {
    let string1 = String::from("abcd");
    let result;
    {
        let string 2 = String::from("xyz");
        result = longest(string.as_str(), string2.as_str());
    }
    println!("The longest string is {result}"); // Error
}
```

Breakdown:
- `string1` is created in the outer scope and lives until the end of `main`.
- `string2` is created in the inner scope and is dropped as soon as the inner scope ends.
- We call `longest`, which returns either `string1` or `string2`.
- If `longest` returns a reference to `string2`, the program would try to use `result` (which points to `string2`), but `string2` has already been dropped!
  - This would be a dangling reference/pointer.

The problem is: **we can't determine, at compile time, whether the returned reference will be valid because we don't know the lifetimes of the inputs (`x` and `y`)**

You might ask: **Why can't we rely on scopes?**

The answer is the compiler doesn't know the relationship between scopes of the references `x` and `y`. For example: 
- `x` might come from a variable that lives for the entire program.
- `y` might come from a temporary variable inside a smaller scope.
- The function's return value depends on runtime logic, so Rust can't tell at compile time which reference will be returned and thus, the lifetime of the returned value.

Without lifetimes, the compiler cannot safely decide which reference the return value is tied to. The function signature:

```rust
fn longest(x: &str, y: &str) -> &str
```

says, "I will return a reference to some `&str`", but it doesn't say whether the returned reference comes from `x` or `y`. Since Rust's borrow checker enforces memory safety, it needs to know:
- "If I return a reference to `x`, does `x` outlive the caller?"
- "If I return a reference to `y`, does `y` outlive the caller?"

Without explicit lifetimes, the compiler doesn't know the answers to these questions.

---

## Lifetime Annotation Syntax

Lifetime annotations don't change how long any of the references live. Rather, they describe the _relationships_ of the liftetimes of multiple references to each other without affecting lifetimes.

This is very similar to functions that accept any type when the signature specified is a generic type parameter, functions can accept references with any lifetime by specifying a generic lifetime parameter.

### Examples

1. A reference to an `i32` without a lifetime parameter.
2. A reference to an `i32` that has a lifetime parameter named `'a`.
3. A mutable reference to an `i32` that also has the lifetime `'a`.

```rust
&i32            // a reference
&'a i32         // a reference with an explicit lifetime
&'a mut i32     // a mutable reference with an explicit lifetime
```

One lifetime annotation by itself doesn't have much meaning because the annotations are meant to tell Rust how generic lifetime parameters of multiple references relate to each other.

## Lifetime Annotations in Function Signatures

Let's continue with the first example with `longest`.

To use lifetime annotations in function signatures, we need to declare the generic *lifetime* parameters inside angle brackets between the function name and the parameter list, just as we did with generic *type* parameters.

We want the signature to express the following constraint:
- The returned reference will be valid as long as both parameters are valid.

This is the relationship between lifetimes of the parameters and the return value.

We'll name the lifetime `'a` and then add it to each reference, as shown below:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```


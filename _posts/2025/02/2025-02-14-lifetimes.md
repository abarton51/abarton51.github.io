---
title: "Rust: Validating References with Lifetimes"
excerpt: Explaining how to validate references with lifetimes in Rust.
permalink: /posts/2025/02/014/lifetimes.md/
author: "Austin B."
---

# Preface

I am *not* an expert in Rust. In fact, I have just recently taken up the endeavor of learning Rust. I am currently finishing my first end-end read of the _The Rust Programming Language_ by Steve Klabnik and Carol Nichols. This book is **fantastic** and I highly recommend it for learning. In addition to this reading, I am using Rust as my language of choice for a numerical analysis course as well as a small TCP chat application I'm working on in my free time.

This page serves as a summary and my own reference of what I learned about _generic lifetimes_ in Rust as well as some other helpful resources for clarification and/or review. Much of the information here is taken directly from _The Rust Programming Language_, but I have modified some sections to expand on various concepts, explain it differently, or omit certain information for the sake of brevity.

Enjoy :crab:

---

# Resources and Prereqs

**Resources:**

- [*The Rust Programming Language*](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)

**Recommended Pre-requisites:**

- [Rust: Ownership](/posts/2025/02/01/rust-ownership/)

---

# Lifetimes

**Lifetimes** are how the compiler keeps track of how long references are valid. _Every reference in Rust has a lifetime, even if it's not explicitly written._

Lifetimes ensure that a reference is never used after the value it points to has been dropped (gone out of scope). This prevents _dangling references_.

# Validating References with Lifetimes

Lifetimes are a kind of _generic_ type in Rust. But instead of ensuring that type has a certain kind of behavior, lifetimes ensure that references are valid as long as we need them to be.

**Every reference in Rust has a *lifetime***.

The **lifetime** is the scope for which that reference is valid.

Most of the time, lifetimes are implicitly inferred, just like most of the time types are inferred. We must annotate types only when multiple types are possible. In a similar way, we must annotate lifetimes when the lifetimes of references could be related in a few different ways.

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

The function signature tells Rust (the compiler and more specifically, the *borrow checker*) that for some lifetime `'a`, the function takes two paramters, both of which are string slices that live at least as long as lifetime `'a`.

The function signature also tells Rust that the string slice returned from the function will live at least as long as lifetime `'a`.

In practice, it means that the lifetime of the reference returned by the `longest` function is the same as the smaller of the lifetimes of the values referred to by the function arguments. These relationships are what we want Rust to use when analyzing this code.

Remember, when we specify the lifetime parameters in this function signature, we are **not changing** the lifetimes of any values passed or returned. Rather, we're specifying that the *borrow checker* should reject any values that don't adhere to these constraints.

Note that the `longest` function doesn't need to know exactly how long `x` and `y` will live, only that some of the scope can be substituted for `'a` that will satisfy this signature.

When annotating lifetimes in functions, the annotations go in the function signature, not in the function body. The lifetime annotations become part of the contract of the function, much like the types in the signature. Having function signatures contain the lifetime contract means the analysis the Rust compiler can be simpler.

When we pass concrete references to `longest`, the concrete lifetime that is substituted for `'a` is the part of the scope of `x` that overlaps with the scope of `y`. In other words, the generic lifetime `a` will get the concrete lifetime that is equal to the smaller of the lifetimes of `x` and `y`.

Here are some examples using references with various concrete lifetimes to understand lifetime annotations and how they restrict the `longest` function.

```rust
fn main() {
    let string1 = String::from("long string is long");
    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {result}");
    }
}
```

- `string1` is valid until the end of the outer scope.
- `string2` is valid until the end of the inner scope.
- `result` references something that is valid until the end of the inner scope.

The borrow checker approves of this code, it compiles, and runs successfully.

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {result}");
}
```

When we compile this code however, we get a compile-time error that states: `'string2' does not live long enough`

The error shows that for `result` to be valid for the `println!` statement, `string2` would also need to be valid until then. This is because Rust we annotated the lifetimes of the function parameters and return values using the same lifetime parameter `'a`.

`result` could be a reference to `string1` which is valid until the end of the outer scope or a reference to `string2` which is valid until the end of the inner scope. We've told the compile though that the lifetme of the reference returned by `longest` is the same of the smaller of the lifetimes of the references passed in.

In this case `result` stores a reference to `string1`, but that isn't determined at compile time. The compiler only knows what we've told it through lifetime annotations. So it essentially assumes that result is a reference with a lifetime that lasts until the end of the inner scope.

## Thinking in Terms of Lifetimes

If instead, we always returned the first argument, we would have the function signature defined as

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

This is because we specify lifetime parameters based on what our function is doing. Since the value returned as no relationship to the argument `y`, we have no need specifying a relationship between the lifetimes of the value returned and `y`.

When returning a reference from a function, the lifetime parameter for the return type needs to match the lifetime parameter for one of the parameters.

If the reference returned does *not* refer to one of the parameters, it must refer to a value created within this function. *However*, this would be a *dangling reference* because the value will go out of scope at the end of the function.

Consider,

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

Here, even though we've specified a lifetime parameter `'a` for the return type, this implementation will fail to compile because the return value lifetime is not related to the lifetime of the parameters at all. Here is the error message:

```
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return value referencing local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ------^^^^^^^^^
   |     |
   |     returns a value referencing data owned by the current function
   |     `result` is borrowed here

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

The problem is that `result` goes of out scope and gets cleaned up at the end of the function. So we are returning a reference to a value that doesn't exist after the function returns!

    Remember: local variables in functions are stored on the stack. When the function returns, the stack is cleaned up.

There is no way we can specify lifetime parameters that would change the dangling reference, and Rust won't let us create a dangling reference.

In this case, the best fix would be to return an owned data type rather than a reference so the calling function is then responsible for cleaning up the value.

## Lifetime Annotations in Struct Definitions

### Recalling Structs

Structs in Rust are custom data types that let you package together and name multiple related values that make up a meaningful group. Each piece of data in a struct can be of a different type, and each piece has a name so it's clear what the values mean.

Structs with owned types are straightforward - they own all their data directly. For example:

```rust
struct User {
    username: String,  // Owned String type
    email: String,     // Owned String type
    active: bool,      // Owned bool type
}
```

We can define structs to  hold references, but in that case we would need to add a lifetime annotation on every reference in the struct's definition.

Example:

```rust
struct ImportantExcerpt<'a> {
    part : &'a str,
}

fn main() {
    let novel = String:: from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().unwrap();
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

This struct has the single field `part` that holds a string slice, which is a reference. As with generic data types, we declare the name of the generic lifetime parameter inside the angle brackets after the name of the struct so we can use the lifetime parameter in the body of the struct definition.

This annotations means an instance of `ImportantExcerpt` can't outlive the reference it holds in its `part` field. That is, the `'a` lifetime on the struct is telling the borrow checker "any reference stored in this struct must be valid for at least as long as the struct itself exists." This ensures that the struct never outlives any of the references it contains, which would create dangling references.


The `main` function here creates an instance of `ImportantExcept` struct that holds a reference to the first sentence of the `String` owned by `novel`. The data in `novel` exists before the `ImportantExcerpt` instance is created. In addition, `novel`, doesn't go out of scope until after the `ImportantExcerpt` goes out of scope, so the reference in the `ImportantExcerpt` instance is valid.

Consider this example:

```rust
fn main() {
    let i;
    {
        let novel = String::from("Call me Ishmael. Some years ago...");
        let first_sentence = novel.split('.').next().unwrap();
        i = ImportantExcerpt {
            part: first_sentence,
        };
    } // novel goes out of scope here
    println!("{}", i.part); // Error! novel was dropped while still borrowed
}
```

This code throws a compile-time error because the lifetime `'a` in `ImportantExcerpt<'a>` tells Rust the the reference stored in `part` must live at least as long as the struct. However, we're trying to keep the struct `i` alive longer than the data it's referencing (`novel`), which violates this guarantee.

## Lifetime Elision

Recall: _Every reference has a lifetime_ and that you need to _specify lifetime parameters for functions or structs that use references_.

However, consider the following example:

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

This code compiles without lifetime annotations. Why is that?

The reason is historical: in early versions of Rust (pre-1.0), this code wouldn't have compiled because every reference needed an explicity lifetime. At that time, the function signature would have been written as 

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

Eventually, the Rust team found that Rust programmers were entering the same lifetime annotations over and over in particular situations. These situations were predictable and had deterministic patterns.

The developers decided to program these patterns into the compiler so that the borrow checker could infer lifetimes in these situations and wouldn't need explicity annotations.

The patterns programmed into Rust's analysis of references are called ***lifetime elision rules***. These aren't rules for programmers to follow. They are a set of particular cases that the compiler will consider, and if your code fits these cases, you don't need to write the lifetimes explicitly.

The elision rules don't provide full inference. If there is amiguity as to what the lifetimes the references have after the rules are applied, the compiler will simply not guess and throw an error at compile time seeking lifetime annotations.

Lifetimes on function or method parameters are called _input lifetimes_, and lifetimes on return values are called _output lifetimes_.

The compiler uses three rules to figure out the lifetimes of the references when there aren't explicity annotations. The first rule applies to input lifetimes and the second and third apply to output lifetimes.

If the compiler gets to the end of the three rules and there are still references for which it can't figure out lifetimes, the compiler will stop with an error. These rules apply to `fn` definitions and `impl` blocks.

Here are the rules:

### First Rule

The compiler assigns a lifetime paramters to each parameter that's a reference. In other words, a function with one parameter gets one lifetime parameter: `fn foo<'a>(x: &'a i32)`; a function with two will get two: `fn foo<'a, 'b>(x: &'a i32, b: &'b i32)` and so on.

### Second Rule

If there is exactly one input lifetime paramter, that lifetime is assigned to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.

### Third Rule

If there are multiple input lifetime paramters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters.

## Lifetime Annotations in Method Definitions

When we implement methods on a struct with lifetimes, we use the same syntax as that of generic type parameters.

Lifetime names for struct fields always neeed to be declared after the `impl` keyword and then used after the struct's name because those lifetimes are part of the struct's type.

In method signatures inside the `impl` block, references might be tied to the lifetime of references in the struct's fields, or they might be independent.

In addition, the lifetime elision rules often make it so that lifetime annotations aren't necessary in method signatures.

Consider the example with `ImportantExcerpt`. First we'll use a method named `level` whose only parameter is a reference to `self` and whose return value is an `i32`, which is not a reference to anything.

```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

The lifetime parameter declaraion after `impl` and its use after the type name are required, but we're not required to annotate the lifetime of the reference to `self` because of the first elision rule.

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {announcement}");
        self.part
    }
}
```

There are two input lifetimes, so Rust applies the first lifetime elision rule and gives both `&self` and `announcement` their own lifetimes.

Then, because one of the parameters is `&self`, the return type gets the lifetime of `&self`, and all lifetimes have been account for.

## Static Lifetime

One special lifetime we need to discuss is `'static`, which denotes that the affected reference _can_ live for the entire duration of the program.

_ALL_ string literals have the `'static` lifetime, which we can annotate as follows:

```rust
let s: &'static str = "I have a static lifetime";
```

The text of this string is stored directly in the program's binary, which is always available. Therefore, the lifetime of all string literals is `'static`.

You might see suggestions to use the 'static lifetime in error messages. But before specifying 'static as the lifetime for a reference, think about whether the reference you have actually lives the entire lifetime of your program or not, and whether you want it to. Most of the time, an error message suggesting the 'static lifetime results from attempting to create a dangling reference or a mismatch of the available lifetimes. In such cases, the solution is to fix those problems, not to specify the 'static lifetime.

## Generic Type Parameters, Trait Bounds, and Lifetimes Together

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {ann}");
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

This is a modification of the `longest` function we used before. However, now it has an extra parameter named `ann` of the generic type `T`, which can be filled in by any type that implements the `Display` trait as specified by the `where` clause.

This extra parameter will be printed using `{}`, which is why the `Display` trait bound is necessary.

Because lifetimes are a type of generic, the declaraions of the lifetime parameter `'a` and the generic type parameter `T` go in the same list in the angle brackets after the function name.

## Summary

A lot was covered here. For many people, this is perhaps the most technically challenging part of learning Rust. However, with this knowledge and some practice, I firmly believe anyone is capable of becoming proficient with lifetimes in Rust.


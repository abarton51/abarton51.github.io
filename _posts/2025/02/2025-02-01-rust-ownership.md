---
title: "Rust: Ownership"
excerpt: Explaining how ownership in Rust works.
permalink: /posts/2025/02/01/rust-ownership.md/
author: "Austin B."
---

# Preface

I am *not* an expert in Rust. In fact, I have just recently taken up the endeavor of learning Rust. I am currently finishing my first end-end read of the _The Rust Programming Language_ by Steve Klabnik and Carol Nichols. This book is **fantastic** and I highly recommend it for learning. In addition to this reading, I am using Rust as my language of choice for a numerical analysis course as well as a small TCP chat application I'm working on in my free time.

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

### Approaches to Managing Memory :thought_balloon:

There are generally two types of programming languages:

- **Garbage Collector (GC)** manages memory :garbage:
  - Java, Python, Go, etc.

- **Programmer** manages memory
  - C, C++, Rust, etc.

#### Rust's Approach

Rust is in the category where the programmer manages memory. *However*, Rust's compiler enforces safe memory management. More specifically, it has a portion of the compiler called the **borrow checker**, which is specifically designed to ensure references to memory are always valid, there are not data races, and that ownership rules are being followed. 

The borrow checker achieves this through analyzing:
- **Ownership**
- **Borrowing** (for later)
- **Lifetimes** (for later)

### Ownership Rules

Let's take a look at the ownership rules. Keep these rules in mind as we work through the examples that illustrate them:

- Each value in Rust has an *owner*.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

### Scope and Ownership :telescope:

A variable is valid within its scope:

```rust
{                       // s is not valid here, because it isn't declared yet
    let s = "yellow";    // s is valid here
}                       // s goes out of scope
```

There are two important points in time here:

- When `s` comes *into* scope, it is valid.
- It remains valid until it goes out of scope.

At this point, the relationship between scopes and when variables are valid is similar to that in other languages.

In this example, `s` owns the `String`. Why this matters and what this means we will explain in more detail in the next section.

### Moving Ownership :dash:

The `String` type manages data allocated on the **heap** and as such is able to store an amount of text that is unknown to us at compile time. This is in contrast to the `str` data type, which is not. There are two things that need to be be taken care when managing memory on the heap:

- The memory must be requested from the memory allocator at runtime.
- We need a way of returning this memory to the allocator when we're done with our `String`.

That first part is done by us: when we call `String::from`, its implementation requests the memory it needs. This is pretty much universal in programming languages.

In languages with a *garbage collector* (GC), the GC keeps track of and cleans up memory that isn't being used anymore, and we don't need to think about it.

In most languages without a GC, it's our responsibility to identify when memory is no longer being used and to call code to explicitly free it, just as we did to request it. 

- Doing this correctly has historically been a difficult programming problem. If we forget, we'll waste memory. If we do it too early, we'll have an invalid variable (such as a _dangling reference_). If we do it twice, that's a bug too (free-after-free). We need to pair exactly one `allocate` with exactly one `free`.

Rust takes a different path: the memory is automatically freed once the variable that *owns* it goes out of scope. Here is the example again with annotations and a visualization to help explain what is happening here:

```rust
{
	let s = String::from("yellow"); // s is valid here
} // s goes out of scope. 
// Rust drops s, freeing the memory allocated on the heap
```

Here is the `String` that `s` is bound to visualized in memory while `s` is in its scope:

```mermaid
graph TD; 
	subgraph String_Contents["yellow: Data on Heap"]
		direction TB
		char_0["0: 'y'"]
		char_1["1: 'e'"]
		char_2["2: 'l'"]
		char_3["3: 'l'"]
		char_4["4: 'o'"]
		char_5["5: 'w'"]
	end
	subgraph String_Type["s: String Structure"]
		direction TB
		ptr["Pointer: 0x1234"] --> char_0
		len["Length: 5"]
		cap["Capacity: 8"]
	end
```

Once out of scope, `s` is invalidated and the heap allocated memory is freed through Rust's RAII (Resource Acquisition Is Initialization) pattern. In C, the programmer would do this with `free()`. However, Rust automatically calls the `drop` function for us, which handles memory cleanup.

When assigning a `String` to another variable, ownership *moves*:

```rust
let s1 = String::from("yellow");
let s2 = s1; // s1 is now invalid
println!("{}", s1); // ERROR: s1 is moved
```

Rust prevents double frees by invalidating `s1`. This is illustrated below with the (MOVED) label.

```mermaid
graph TD; 
	subgraph String_Contents["yellow"]
		direction TB
		char_0["0: 'y'"]
		char_1["1: 'e'"]
		char_2["2: 'l'"]
		char_3["3: 'l'"]
		char_4["4: 'o'"]
		char_5["5: 'w'"]
	end
	subgraph String_Type_1["s1 (MOVED)"]
		direction TB
		ptr_1["Pointer: 0x1234 (moved)"]
		len_1["Length: 5 (moved)"]
		cap_1["Capacity: 8 (moved)"]
	end

	subgraph String_Type_2["s2"]
		direction TB
		ptr_2["Pointer: 0x1234"] --> char_0
		len_2["Length: 5"]
		cap_2["Capacity: 8"]
	end
```

Notice that `s1` still exists and is bound to the same data that `s2` is within its corresponding scope, but the *ownership* of the data is *moved* from `s1` to `s2`. This invalidates usage of `s1` since only one owner is allowed.

The data on the heap still has *one owner* - `s2` and when `s2` (not `s1`) goes out of scope, Rust can safely deallocate the memory on the heap without worrying about any bugs since any previous owners, such as `s1`, have moved their ownership and are invalid.

If Rust allowed both of these variables to have ownership, there are possibilities of dangling references, use-after-frees, double frees, and just having data that was not expected.

If you’ve heard the term _shallow copy_ and _deep copy_ while working with other languages, the concept of copying the pointer, length, and capacity without copying the data probably sounds like making a shallow copy. The only difference here is that Rust tracks ownership in order to safely and properly free heap allocated memory.

### 4. Cloning Data :robot:

If a deep copy is needed, use `.clone()`.

This creates a second `String` the is a *clone* of the first one. That is, two separate and distinct memory allocated forms of data that contain the same contents that it points to. 

```rust
let s1 = String::from("yellow");
let s2 = s1.clone();
println!("{} and {}", s1, s2); // Both are valid
```

```mermaid
graph TD; 
	subgraph String_Contents_1["yellow"]
		direction TB
		char_0_1["0: 'y'"]
		char_1_1["1: 'e'"]
		char_2_1["2: 'l'"]
		char_3_1["3: 'l'"]
		char_4_1["4: 'o'"]
		char_5_1["5: 'w'"]
	end
	subgraph String_Type_1["s1"]
		direction TB
		ptr_1["Pointer: 0x1234"] --> char_0_1
		len_1["Length: 5"]
		cap_1["Capacity: 8"]
	end

	subgraph String_Contents_2["yellow"]
		direction TB
		char_0_2["0: 'y'"]
		char_1_2["1: 'e'"]
		char_2_2["2: 'l'"]
		char_3_2["3: 'l'"]
		char_4_2["4: 'o'"]
		char_5_2["5: 'w'"]
	end

	subgraph String_Type_2["s2"]
		direction TB
		ptr_2["Pointer: 0x2345"] --> char_0_2
		len_2["Length: 5"]
		cap_2["Capacity: 8"]
	end
```

Note that the pointers in `s1` and `s2` are different. Cloning allocates data on the heap, copies the data to that location, and creates a new `String` structure pointing to that data. `s1` maintains ownership because it is still the sole owner of its data and `s2` is the sole owner of its data.

### 5. Copy Trait

Types that have a known size at compile time and don't require special cleanup when dropped can implement the `Copy` trait, allowing them to be duplicated rather than moved:

```rust
let x = 5;
let y = x; // x is still valid because integers implement Copy
```

This works for all primitive types as all primitive types in Rust implement `Copy` and are stored on the **stack**.

### 6. Ownership and Functions :passport_control:

Passing a variable to a function moves ownership:

```rust
fn takes_ownership(s: String) {
    println!("{}", s);
} // s is dropped here

let s = String::from("hello");
takes_ownership(s);
println!("{}", s); // ERROR: s is moved
```

This might seem weird, but let's think about why this is a problem. After `take_ownership(s)`, if we accessed `s`, how are we to know what `s` is or if it's anything at all? **We don't** (or at least, that's what we assume).

This feature of ownership is a result of Rust's compiler dropping and invalidating variables to prevent bugs like double-frees. If this code above was valid, pesky bugs like that can exist.

Returning a value transfers ownership back:

```rust
fn gives_ownership() -> String {
    String::from("hello")
}

let s = gives_ownership(); // s now owns the value
```

Remember the previous code that gave an error? Well we can solve it with this:

```rust
fn takes_ownership(s2: String) {
    println!("{}", s2);
    s2
} // s2 is dropped here

let s1 = String::from("hello");
let s2 = takes_ownership(s1);
println!("{}", s2); // Yay, it works!
```

This seems redundant and a bit annoying, and in truth it is. We could've made a clone with `s.clone()` instead also to pass to `takes_ownership`. The `takes_ownership` function doesn't *need* to own the `String`.

This is where **references** come in. Instead of having the function take ownership of the variable, it an *borrow* the variable by taking a *reference* to the variable.

With understanding ownership now, you'll be well on your way with references! :smile:

---

# Summary

Rust provides memory safety without a garbage collector, making it both efficient and reliable. 

Its ownership system ensures safe memory management through strict rules: each value has a single owner, and ownership transfers prevent invalid memory access. Understanding ownership is crucial for writing Rust code effectively.

Now that you know Rust’s core principles and how ownership works, you can start coding! 

To create a new Rust project, which we refer to as a **crate**, simply run:
-  `cargo new <project_name>`

You can run the crate by running:
- `cargo run`

In Rust, the compiler is your friend (just an uptight one). Rust has lots of features to teach and guide you. You can even get detailed explanations of error messages using:
- `rustc --explain`

The official [Rust Book](https://doc.rust-lang.org/book/title-page.html) and [Rustlings](https://github.com/rust-lang/rustlings) exercises are great next steps. Soon enough you'll consider yourself an experienced Crustacean. Happy coding!

## References

1. Klabnik, S., & Krycho, C. (n.d.). _The rust programming language_. The Rust Programming Language - The Rust Programming Language. https://doc.rust-lang.org/book/title-page.html
2. Stack Overflow Community. (1960, August 1). _What does rust have instead of a garbage collector?_. Stack Overflow. https://stackoverflow.com/questions/32677420/what-does-rust-have-instead-of-a-garbage-collector
3. _Rustlings_. rustlings. (n.d.). https://rustlings.cool/


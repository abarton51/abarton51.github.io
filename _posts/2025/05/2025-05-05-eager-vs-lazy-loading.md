---
title: "Eager vs. Lazy Loading"
publishedAt: "2025-05-05"
excerpt: "A moderately in-depth lesson on eager versus lazy loading and evaluation."
permalink: /posts/2025/05/05/eager-vs-lazy-loading.md/
author: 'Austin Barton'
---

This is an instructive overview with real examples that demonstrate how eager and lazy loading work and when to use each. This post starts at the high conceptual level and eventually makes its way down to understanding the system-level implications.

To demonstrate more comprehensive usages of lazy loading and evaluation, we compare [Pandas](https://pandas.pydata.org/) and [Polars](https://docs.pola.rs/api/python/stable/reference/index.html) and examine Polars's DataFrame and LazyFrame objects.

## Introduction

In this lesson, we’ll explore the difference between eager and lazy loading/evaluation — two fundamental strategies for resource management in software development.

We'll use Rust as our language of choice to learn lazy loading. Part of the reason we are using Rust here is because:
- Rust is a "low-level" language and thus, it makes you explicitly manage memory and execution flow, helping to build a deeper understanding of how and when data is loaded, computed, and accessed.
- **Polars** is a Python data manipulation library written in Rust. Later in this blog post, we will use the Pandas and Polars libraries to illustrate eager and lazy loading examples.

Very briefly, we will explain the technical differences between evaluation and loading, but the pattern of eager vs. lazy is what matters, and many times evaluation and loading can refer to and include one another. More generally, you could say eager vs. lazy _execution_ (a bit of semantics, but it helps to be precise!).

---

## Loading and Evaluation

### What is "Loading"?

_Loading_ refers to retrieving or initializing external data or resources, such as reading from disk, fetching over a network, or decoding a file.
- Fetching a file.
- Connecting to a database.
- Parsing an image from bytes on a disk.
- And more.

### What is "Evaluation"?

_Evaluation_ refers to computing results from already-loaded or already-available data.
- Filtering rows of a DataFrame.
- Aggregating data (`sum`, `mean`, etc.).
- Mapping functions over iterators

### Two Strategies to Loading and Evaluation

There two primary strategies:

- **Eager**: Load/evaluate everything upfront, regardless of whether it is needed immediately.
- **Lazy**: Defer loading/evaluation until it is actually needed.

NOTE: From here on out, we will use focus on loading but in some cases we will use the term "loading" as a blanket term to refer to evaluation of certain transformations on the data as well.

---

## Loading

### Eager Loading

**Definition**

Eager loading means computing or loading a resource immediately, as part of the initial setup.

**Analogy**

Imagine you go to a buffet and fill your plate with everything just in case you might want it.

**Pros**
- Simple code that's easy to read: No need to check if it's initialized.
- Good when you know the data will definitely be used.
- Lower runtime overhead during actual usage.

**Cons**
- Potentially wasteful (memory, compute)
- Longer startup time.
- Not feasible with large amounts of data: Requires loading everything at once into memory.

### Lazy Loading

**Definition**

Lazy loading delays the initialization or computation of a resource until the first time it’s accessed.

**Analogy**

You're at a buffet but only get up to get food when you're actually hungry for that specific dish.

**Pros**
- Faster startup.
- More memory and compute efficient.
- Great for rarely-used or optional resources.

**Cons**
- Slightly more complex implementation.
- First access can incur a performance hit.
- Must be carefully handled in multi-threaded programs.

---

## Eager vs. Lazy: Walkthrough in Rust

Rust makes it easy to express both patterns safely and efficiently.

We will walk through:
- Eager loading via regular struct construction.
- Lazy loading with `Option`.
- Lazy loading with `OnceCell` and `Lazy` (from the `once_cell` crate).

### Eager Loading Example

```rust
struct Config {
	contents: String,
}

impl Config {
	fn new(file_path: &str) -> Self {
		let contents = std::fs::read_to_string(file_path)
			.expect("Failed to read file");
		Config { contents }
	}

	fn get_contents(&self) -> &str {
		&self.contents
	}
}

fn main() {
	let config = Config::new("settings.txt"); // <- Entire resource is loaded here.
	println!("Config: {}", config.get_contents());
}
```

The file is read as soon as we call `Config::new()`, regardless of whether you end up using the contents.

### Lazy Loading Example: Manual with `Option`

```rust
struct Config {
	file_path: String,
	contents: Option<String>,
}

impl Config {
	fn new(file_path: &str) -> Self {
		Config {
			file_path: file_path.to_string(),
			contents: None,
		}
	}

	fn get_contents(&mut self) -> &str {
		if self.contents.is_none() {
			let contents = std::fs::read_to_string(&self.file_path)
				.expect("Failed to read file");
			self.contents = Some(contents);
		}
		self.contents.as_ref().unwrap()
	}
}

fn main () {
	let mut config = Config::new("settings.txt");
	println!("Config: {}", config.get_contents()); // <- loading happens here, when we actually need it.
}
```

The file is only read when get_contents() is first called. Subsequent calls reuse the cached result.

### Lazy Initialization Example: `OneCell`

Rust's once_cell crate provides safe one-time initialization for lazy values.

```rust
use once_cell::unsync::OnceCell;

struct Config {
	file_path: String,
	contents: OnceCell<String>,
}

impl Config {
	fn new(file_path: &str) -> Self {
		Config {
			file_path: file_path.to_string(),
			contents: OnceCell::new(),
		}
	}

	fn get_contents(&self) -> &str {
		self.contents.get_or_init(|| {
			std::fs::read_to_string(&self.file_path)
				.expect("Failed to read file")
		})
	}
}

fn main() {
	let config = Config::new("settings.txt");
	println!("Config: {}", config.get_contents()); // Lazy and thread-safe
}
```

You can also use `once_cell::Lazy` for global initialization (i.e., static variables).

### When to Use What?

| Feature             | Eager Loading                                | Lazy Loading                                         |
|---------------------|----------------------------------------------|------------------------------------------------------|
| Simplicity          | Simple and easy to reason about              | Slighlty complex                                     |
| Startup Time        | Slow if data is unused                       | Fast, loads only when needed                        |
| Runtime Performance | Fast if resource is frequently accessed      | First access can be slower (more overhead required usually) |
| Memory Efficiency   | May use unnecessary memory, limited to memory available | On-demand memory usage, not as limited to memory available |
| Thread Safety       | Safe and straightforward                     | Requires synchronization or `OnceCell`              |

With lazy loading, there are A LOT of optimizations that one can implement in addition. In the next section, we are going to demonstrate this using the Polars data manipulation library in Python.

---

## Pandas vs. Polars

**Pandas** is the de facto standard for data manipulation in Python. It uses **eager execution**: when you load a file or perform an operation, it happens immediately in memory.

While this makes the API intuitive and results immediate, it doesn't scale well for large datasets, as it eagerly loads all data into memory and performs every operation as defined.

**Polars**, on the other hand, is a modern data manipulation library in Python that is written in Rust. One of its core innovations is that it supports **both eager and lazy execution** modes:
- `pl.DataFrame`: Eager execution, similar to Pandas.
- `pl.LazyFrame`: Lazy execution. Operations are deferred and compiled into a query plan, only executing when `.collect()` is called, which returns a `DataFrame`.

### Pandas: Eager Loading

```python
import pandas as pd
df = pd.read_csv("huge.csv")
```

- The entire file is loaded into memory at once.
- A `DataFrame` object is constructed - with full allocation of columns and rows.
- Every transformation such as `.filter(...)`, `.groupby(...)`, etc. is executed immediately, creating intermediate copies unless done in-place.

**Implications:**

- Fast for small data (less overhead).
- Simple mental model.
- Memory-hungry for large files (all of it must fit in RAM)
- Intermediate resources can cause extra allocations and Garbage Collector (GC) pressure.
- High memory usage even for operations that only need a few columns.

### Polars: Lazy Loading

```python
import polars as pl
lf = pl.scan_csv("huge.csv")

```

- The file is not read immediately.
- Instead, it returns a `LazyFrame`, which is essentially a query plan.
- You can now compose transformations with evaluating them.

```python
result = (
	pl.scan_csv("huge.csv")
		.filter(pl.col("age") > 30)
		.group_by("city")
		.agg(pl.col("salary").mean())
		.collect() # <-- execution happens here, returns a DataFrame
)
```

---

## Lazy and Eager Loading with Polars

Polars has both DataFrame and LazyFrame classes. DataFrames are essentially the actual structured data realized in memory, whereas a LazyFrame is a lazy execution _plan_ to create a DataFrame.

DataFrames in Polars are essentially the same as the DataFrames we've seen in Pandas.

### LazyFrames

Here is documentation describing the LazyFrame class taken from the [source code](https://github.com/pola-rs/polars/blob/py-1.29.0/py-polars/polars/lazyframe/frame.py#L220-L8136):
- "Representation of a Lazy computation graph/query against a DataFrame. This allows for whole-query optimisation in addition to parallelism, and is the preferred (and highest-performance) mode of operation for polars."


| Feature                  | LazyFrame                                 | DataFrame                            |
|--------------------------|-------------------------------------------|--------------------------------------|
| Evaluation Strategy      | Deferred (lazy)                           | Immediate (eager)                    |
| Memory Usage             | Minimal - optimized (columnar projection) | Allocates memory immediately         |
| Execution Time           | On `.collect()`                           | When transformations are defined     |
| Optimization             | Query planned and optimized               | No query plan                        |
| Use Case                 | Large data, scalable pipelines            | Small to medium data                 |
| API Surface              | Similar API, designed for chaining        | More direct data manipulation        |
| Performance (large data) | Very efficient, streaming-friendly        | Bottlenecks on RAM and CPU cache     |

### Example: `scan_csv` vs. `read_csv`

```python
lf = pl.scan_csv("huge.csv")
result = (
	lf
	.filter(pl.col("status") == "active")
	.select(["name", "email"])
	.collect() # Only now it runs
)

df = pl.read_csv("huge.csv") # Reads everything into memory
result = df.filter(
	pl.col("status") == "active")
	.select(["name", "email"]) # Executed immediately, no query plan created
```

- `scan_csv`: returns a `LazyFrame`, streams the file, reads only necessary columns and row.
- `.collect()`: executes the query plan and returns the resulting `DataFrame`.

### Details of Lazy Loading in Polars with Streaming and JIT

Polars' lazy execution engine builds a deferred computation plan rather than executing operations immediately. This allows it to:

- Optimize the entire query plan before execution.
- Stream data in batches to reduce memory pressure.
- Avoid redundant computations or data loads.
- Leverage just-in-time (JIT) style deferred execution where nothing happens until `.collect()` is called.

### How the Execution Pipeline Works

1. **Query Planning**
- Operations are chained onto a `LazyFrame`.
- Internally, Polars constructs a logical plan as a **directed acyclic graph (DAG)** of transformations.
- No actual computation or data access occurs yet.

2. **Query Optimization**
- Polars performs predicate pushdown, projection pushdown, and simplification:
	- Predicate pushdown: Filters are pushed as close as possible to the data source.
	- Projection pushdown: Only columns used in downstream operations are loaded.
	- Common subexpression elimination, type coercion, etc., also happen here.

3. **Chunked Streaming Execution**
- If streaming mode is supported and enabled (either explicitly or via .collect_streaming()), Polars processes data in batches:
	- Reads and parses CSV/parquet files in row groups or chunks.
	- Applies the logical plan incrementally per batch.
	- Intermediate results are discarded or aggregated without keeping the full dataset in memory.

4. **Memory Efficient Aggregation**

- Aggregations like `.group_by(...).agg(...)` maintain partial states across streamed chunks.
- For example, `mean` is computed using running totals and counts.
- The full table is never loaded — just the minimal state necessary for the final result.

5. **Materialization on `.collect()`**
- Once .collect() is called, the optimized query is executed and the final eager DataFrame is assembled.
- Only this final result is materialized in memory.

---

## A Conceptual Peek Under The Hood: System-Level Intuition

### What's Happening Under the Hood?

When we say "eager" vs. "lazy", we're not just talking about the order of execution. We're talking about how the code interacts with memory, disk, CPU caches, system resources, and the compiler/runtime's optimization strategies.

These design choices deeply impact performance, memory efficiency, and scalability for large datasets.

### Conceptual Model: Memory-Execution Hierarchy

Imagine your computer has a hierarchy of memory and execution stages:

```text
[Disk] -> [RAM] -> [L1/L2 CPU Cache] -> [CPU Registers] -> [ALU]
```

- Disk: Slow, persistent. Holds CSVs, Parquet files, etc.
- RAM: Fast, volatile. Where your program keeps runtime data.
- CPU Cache: Tiny, very fast. Where hot data is preloaded for performance.
- Registers: Super tiny, closest to ALU. Where data is stored to be used by ALU.

**Eager loading**
- Pulls the entire dataset from disk → RAM → cache upfront.
- All operations are executed as soon as they're defined.
- Risk: blowing through RAM and CPU caches unnecessarily.

**Lazy loading**
- Defers all data reads and computations until absolutely necessary.
- Uses query planning, chunking, and streaming to minimize memory pressure.
- Only needed columns and rows are materialized in memory, when needed.

### Paging, Caching, and Hardware

When your program loads large datasets:

- **Disk I/O** is the bottleneck → lazy helps by reducing total bytes read.

- **CPU caches** (L1/L2) are small — eager execution can blow them out.

- **Page faults**: Eager loads often cause the OS to allocate more virtual memory pages at once. Lazy loading touches fewer pages and reduces page faults.

- **TLB misses**: Lazily accessed memory is better localized (especially columnar data), reducing cache misses and Translation Lookaside Buffer (TLB) overhead.

| Aspect                    | Eager                                | Lazy                                         |
|---------------------------|---------------------------------------|----------------------------------------------|
| Memory usage              | High                                  | Low to moderate (streamed)                   |
| CPU cache efficiency      | Low (blows cache)                     | High (chunked, columnar,)                    |
| Page faults               | High (entire structure loaded)        | Lower (less virtual memory touched)          |
| GC pressure (GC langs e.g., Python) | High                         | Lower                                        |
| Startup latency           | High (loads all)                      | Low (does nothing until needed)              |
| File I/O                  | Full file read                        | Buffered, filtered, optimized                |

---

## Wrapping Up

- **Eager loading** is simple and fast after initialization, but not memory-efficient for large or optional data.

- **Lazy loading** is more efficient in memory and computation but requires deferred logic and care in multithreaded contexts. Although it has more overhead, there _a lot_ of optimizations you can do with lazy evaluation.

Rust provides tools like:
- `Option` for manual lazy init.
- `OneCell` and `Lazy` for safe, one-time evaluation.
- Lazy iterators by default via the `Iterator` trait (e.g., `.map()`, `.filter()`, etc.).

Polars is one of many great examples of the power of lazy loading in the real world. It's still in its early phases, but I expect to see it start make its way to the main stage for data analysis and manipulation over Pandas.

Hope you enjoyed, cheers! :beers:


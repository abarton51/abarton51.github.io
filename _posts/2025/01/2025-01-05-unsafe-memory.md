---
title: "Unsafe Memory"
excerpt: Overview of concepts involving memory management and bugs due to unsafe memory.
permalink: /posts/2025/01/01/unsafe-memory.md/
author: "Austin B."
---

# Unsafe Memory

Even in the absence of concurrency, unsafe memory operations can lead to serious issues like program crashes, data corruption, or security vulnerabilities. 

Unsafe memory access typically arises from bugs in handling pointers, allocation, and/or deallocation of memory.

---

## Who is in Charge of Memory?

There are generally two types of programming languages:
- Garbage Collector (GC) manages memory
  - Java, Python, Go, etc.
- Programmer manages memory
  - C, C++, Rust, etc.

### Garbage Collector (GC)

A **garbage collector** is an automatic memory management tool that identifies and frees unused or unreachable memory. It tracks memory allocations and determines when they are no longer needed by the program, either through reference counting or detecting objects that are no longer reachable from the program's active memory.

Garbage collection requires additional data structures for tracking memory usage which incurs overhead in:
- **Performance**: Periodic "pauses" for memory cleanup.
- **Memory**: Additional memory is used to store metadata and manage allocations.
- **Complexity**: The language runtime must include the GC system.

### Why Not Use a Garbage Collector?

There are many cases where garbage collection is not suitable:
- **High-performance systems**
- **Minimal resource environment**
- **Specialized memory management needs**

In languages like C, the programmer has complete control over memory management, meaning the programmer has the flexibility to write programs for more specialized requirements. The tradeoff is that the programmer must manually allocate and deallocate memory, which can lead to **unsafe memory bugs** such as:
- **Dangling references/pointers**: A reference/pointer that points to memory that has been freed.
- **Memory leaks**: Forgetting to free memory.
- **Double-free errors**: Freeing the same memory twice.

_We will go over unsafe memory bugs in detail later._

### Rust's Approach

Rust is also in the category where the programmer manages memory. However, Rust's compiler enforces safe memory management. More specifically, it has a portion of the compiler called the **borrow checker**, which is specifically designed to ensure references to memory are always valid, there are not data races, and that ownership rules are being followed. 

The borrow checker achieves this through:
- **Ownership**
- **Lifetimes**

I have posts explaining each of these concepts in detail which are based on _The Rust Programming Language_ book by Steve Klabnik and Carol Nichols. I highly recommend it and you can read the book for free online [here](https://doc.rust-lang.org/book/).

---

Now that we've introduced some concepts about how langauges manage memory, let's go over some bugs in detail :bug:.

## Unsafe Memory Bugs

### 1. **Dangling References**

A **dangling reference** (or **dangling pointer**) occurs when a pointer or reference continues to exist after the memory it points to has been deallocated or freed. Accessing such memory leads to undefined behavior because the memory may now contain garbage values or be reassigned for other purposes.

#### **How it happens**:
- Memory is deallocated, but a pointer or reference to it still exists.
- The pointer is dereferenced after the memory has been freed.

#### **Example** (C++):
```cpp
int* ptr = new int(42); // Dynamically allocate memory
delete ptr;             // Free the allocated memory

int x = *ptr;           // Undefined behavior: ptr is now a dangling pointer
```

#### **Consequences**:
- **Undefined behavior**: Accessing a dangling pointer can cause crashes or corrupt memory. The fact is we don't know what it will do, but the memory location it's pointing to has data that isn't guaranteed to exit.
- **Security vulnerabilities**: Attackers can exploit dangling pointers to access sensitive data or execute arbitrary code. This is called *buffer overflow* and we will go over this in more detail later.

---

### 2. **Double Free**

A **double free** occurs when a program attempts to free the same memory block more than once. This can corrupt the memory allocator's internal structures and lead to crashes or vulnerabilities, such as **use-after-free** attacks.

#### **How it happens**:
- The `free()` or `delete` function is called multiple times on the same pointer.

#### **Example** (C):
```c
int* ptr = malloc(sizeof(int)); 
free(ptr);   // Free the memory
free(ptr);   // Double free: undefined behavior
```

#### **Consequences**:
- **Memory corruption**: The memory allocator's metadata can be corrupted.
- **Security risks**: Attackers can exploit double free vulnerabilities to manipulate memory.

---

### 3. **Use-After-Free**

A **use-after-free** error occurs when a program accesses memory after it has been deallocated. This is a specific case of accessing dangling references.

#### **How it happens**:
- A pointer to deallocated memory is used for reading or writing data.

#### **Example** (C):
```c
int* ptr = new int(42);
delete ptr;         // Memory is freed

*ptr = 10;          // Undefined behavior: use-after-free
```

#### **Consequences**:
- **Program crashes**: Dereferencing invalid memory can lead to segmentation faults.
- **Security vulnerabilities**: Attackers can exploit use-after-free bugs to inject malicious data or code.

---

### 4. **Buffer Overflows**

A **buffer overflow** occurs when a program writes more data to a memory buffer than it can hold, overwriting adjacent memory. This is a common source of security vulnerabilities and is often exploited for attacks like **stack smashing** or **heap spraying**.

#### **How it happens**:
- A program fails to check the size of input data before writing it into a buffer.

#### **Example** (C):
```c
char buffer[10];
strcpy(buffer, "This string is way too long!"); // Overflow
```

#### **Consequences**:
- **Memory corruption**: Adjacent memory is overwritten, leading to unpredictable behavior.
- **Security risks**: Attackers can overwrite return addresses or function pointers to execute arbitrary code.

---

### 5. **Invalid Memory Access**

Invalid memory access occurs when a program attempts to access uninitialized memory or memory it does not own. This includes reading from or writing to out-of-bounds addresses.

#### **How it happens**:
- Accessing memory that hasn’t been initialized.
- Using array indices that are out of bounds.

#### **Example** (C):
```c
int array[5] = {1, 2, 3, 4, 5};
int value = array[10]; // Out-of-bounds access: undefined behavior
```

#### **Consequences**:
- **Program crashes**: Segmentation faults or memory access violations.
- **Security vulnerabilities**: May leak sensitive data or enable attacks.

---

### 6. **Memory Leaks**

A **memory leak** occurs when dynamically allocated memory is never freed, causing the program to consume increasing amounts of memory over time.

#### **How it happens**:
- A program forgets to free memory before losing all references to it.

#### **Example** (C):
```c
int* ptr = malloc(sizeof(int));
// Memory allocated but never freed
```

#### **Consequences**:
- **Resource exhaustion**: The program or system may run out of memory.
- **Performance degradation**: Excessive memory usage can slow down the system.

---

### How to Avoid Unsafe Memory Issues

1. **Use High-Level Languages**: Use languages like Rust, Python, or Java, which have built-in memory safety features.
    - **Rust** enforces memory safety through ownership, borrowing, and lifetimes.
    - High-level languages like Python automatically manage memory via garbage collection.

2. **Enable Compiler Checks**: Use tools like `valgrind` or `AddressSanitizer` to detect memory errors in C/C++ programs.

3. **Practice Safe Programming**:
    - Initialize variables before use.
    - Avoid manual memory management whenever possible.
    - Use smart pointers like `std::unique_ptr` or `std::shared_ptr` in C++.

4. **Apply Static and Dynamic Analysis**:
    - Use static analyzers to catch potential bugs during compilation.
    - Run dynamic tests to detect memory issues at runtime.

---

### Summary Table of Common Unsafe Memory Issues

| **Issue**               | **Description**                                                                                     | **Example**                       | **Impact**                      |
|-------------------------|----------------------------------------------------------------------------------------------------|-----------------------------------|----------------------------------|
| **Dangling Reference**  | Accessing memory after it has been deallocated.                                                   | Dereferencing freed pointers.     | Crashes, undefined behavior.    |
| **Double Free**         | Freeing the same memory block more than once.                                                     | Calling `free()` twice.           | Memory corruption, crashes.     |
| **Use-After-Free**      | Using memory after it has been deallocated.                                                       | Writing to freed memory.          | Crashes, security vulnerabilities.|
| **Buffer Overflow**     | Writing beyond the bounds of a buffer.                                                            | Overwriting adjacent memory.      | Memory corruption, security risks.|
| **Invalid Memory Access** | Accessing uninitialized or out-of-bounds memory.                                                 | Reading from invalid addresses.   | Crashes, undefined behavior.    |
| **Memory Leak**         | Forgetting to free dynamically allocated memory.                                                  | Allocating without freeing.       | Resource exhaustion, slowdown.  |

---

Unsafe memory access poses significant risks even in single-threaded programs. By understanding these issues and adopting best practices, programmers can minimize the likelihood of bugs and vulnerabilities, leading to safer and more robust software.

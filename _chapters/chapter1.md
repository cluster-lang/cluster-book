---
layout: chapter
title: "Cluster-lang: Chapter 1 — The Basics"
permalink: /chapter1/
prev_chapter: /chapter0/
prev_title: "Chapter 0 — The Cluster Manifesto"
next_chapter: /chapter2/
next_title: "Chapter 2 — Control Flow & Functions"
---

# Chapter 1: The Basics — Core Syntax & Type Primitives

Welcome to the foundational syntax of the **Cluster Programming Language** (`.cl`). Cluster combines the clean, expressive readability of Python with the execution speed of C++ and the compile-time safety of modern systems languages.

---

## 1. Variables & Immutability

In Cluster, variables are **immutable by default** to eliminate accidental mutation bugs and enable aggressive LLVM register optimizations.

### Immutable Declarations (`:=`)
Use the walrus operator `:=` for type-inferred immutable variable bindings:
```cluster
x := 42                // Inferred as 64-bit int
name := "Alice"        // Inferred as string
is_valid := true       // Inferred as bool
pi := 3.14159          // Inferred as 64-bit float
```
Once declared, an immutable variable cannot be reassigned:
```cluster
x := 42
x = 50                 // Compile-time error: Cannot reassign immutable variable 'x'
```

### Mutable Declarations (`let mut`)
When a variable needs to be updated across loops or calculations, declare it explicitly with `let mut`:
```cluster
let mut counter: int = 0
counter = counter + 1

let mut buffer: string = ""
buffer = buffer + "chunk"
```

### Constants (`const`)
Compile-time constants are evaluated during compiler front-end optimization:
```cluster
const MAX_PACKET_SIZE: int = 65536
const PROTOCOL_VERSION: string = "v1.0.0"
```

---

## 2. Primitive Data Types

Cluster provides clean, hardware-aligned 64-bit primitive types:

| Type | Bit Width | Description | Machine Equivalent |
| :--- | :--- | :--- | :--- |
| `int` | 64-bit | Signed two's-complement integer | `int64_t` (CPU register) |
| `float` | 64-bit | IEEE 754 double-precision floating point | `double` (FPU register) |
| `string` | Dynamic | UTF-8 / ASCII byte array with length tracking | High-speed memory buffer |
| `bool` | 1-bit / 8-bit | Boolean logic (`true` or `false`) | `i1` / `uint8_t` |
| `void` | 0-bit | Unit return type for functions with no output | Function return `void` |

---

## 3. High-Performance Vectors (`vector[T]`)

Dynamic arrays in Cluster are first-class, memory-contiguous structures known as `vector[T]`.

### Declaration & Initialization
```cluster
// Empty vectors
let mut numbers: vector[int] = []
let mut names: vector[string] = []

// Vector literal initialization
scores: vector[int] = [98, 85, 92, 100]
```

### Vector Operations
```cluster
let mut items: vector[string] = []

// Append an element (amortized O(1))
items.push("apple")
items.push("banana")

// Get length
count := len(items)    // Returns 2

// 0-indexed element access
first := items[0]      // "apple"

// In-place mutation
items[1] = "blueberry"
```

---

## 4. Strings & Byte-Level Manipulation

Because Cluster is a systems language built for high-speed protocols (as demonstrated in `cluster-curl` and `cluster-redis`), it features fast, zero-copy string inspection routines:

```cluster
text := "Cluster Engine"

// Get length in bytes
total_bytes := len(text)             // 14

// Read unsigned byte value at offset (O(1))
first_byte := byte_at(text, 0)       // 67 (ASCII 'C')

// Extract substring: substring(source, start_index, length)
sub := substring(text, 0, 7)         // "Cluster"

// Convert ASCII byte code back to character string
ch := chr(65)                        // "A"

// String concatenation
greeting := "Hello, " + text + "!"
```

---

## 5. Standard Output & Formatting

Cluster provides high-speed standard I/O primitives:

```cluster
// Print with an automatic newline
println("System initialized successfully.")

// Print without a newline
print("Loading: ")
print("100%\n")

// Automatic string interpolation via str()
count := 42
println("Total matches: " + str(count))
```

---

## 6. Type Conversions & Casting

Type conversions in Cluster are explicit to avoid silent truncation bugs:

```cluster
// String to Integer
num := to_int("12345")               // Inferred as int

// Integer to String
s := str(12345)                      // Inferred as string

// Float to String
f_str := str(3.14159)
```

---

## 7. Architectural Advantages: Why Cluster Beats Legacy Languages

| Feature | C / C++ | Python | Rust | Cluster |
| :--- | :--- | :--- | :--- | :--- |
| **Execution Speed** | Fast (Native) | Slow (10x-100x slower) | Fast (Native) | **⚡ Native LLVM -O3** |
| **Memory Safety** | ❌ None (Segfaults) | Safe (GC pause) | Safe (Borrow checker) | **✅ Memory Safe by Default** |
| **Dependencies** | ❌ Make, CMake, vcpkg | ❌ pip, wheels, venv | ❌ Cargo, 100+ crates | **✅ 100% Dependency-Free** |
| **Syntax Complexity** | High (Pointers, macros) | Minimal | High (Lifetimes, traits) | **✅ Clean, Pythonic Readability** |
| **Binary Output** | Standalone ELF/EXE | Requires Interpreter | Heavy binaries | **✅ Compact, Self-Contained ELF** |

---

## 8. Summary

In this chapter, you learned:
- How to declare immutable bindings (`:=`) and mutable variables (`let mut`).
- Core primitive data types and hardware-level sizes.
- How to manage dynamic contiguous memory with `vector[T]`.
- How to perform byte-level slicing and string manipulation.

In the next chapter, we will explore **Control Flow & Functions** to build structured, modular logic.

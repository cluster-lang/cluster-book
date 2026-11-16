# The Cluster Book: Official Language Guide & Architecture Specification

> [!IMPORTANT]
> **Project Status: Active Development**  
> This documentation is currently under active development. Developers, researchers, and AI agents are invited to read the chapters, test language examples, and contribute! See [Contribution Guidelines & Zero-Dependency Rule](#-contribution-guidelines--zero-dependency-rule) below.

**Author:** mohammadzaid/(cluster) <zkalamgir@proton.me>  
**Organization:** [cluster-lang](https://github.com/cluster-lang)  
**License:** Cluster Community & Fair-Source License (CCFL v1.0 / MIT-Conditional)  

Welcome to **The Cluster Book**, the comprehensive reference manual and guide to the **Cluster Programming Language** (`.cl`).

Cluster is a high-performance compiled systems programming language combining Pythonic syntax readability, compile-time memory safety without a garbage collector, and native execution speed powered by LLVM `-O3`.

---

## 📖 Chapter Roadmap (32 Chapters)

The book covers everything from fundamental syntax to deep compiler internals and standard libraries:

### Part 0: The Vision & Manifesto
- **Chapter 0: The Cluster Manifesto** — Why build Cluster? The dependency horror crisis (99% dead code bloat), breaking the legacy "don't touch running code" dogma, expressive density proof, and the all-in-one vision.

### Part I: Core Language & Control Flow
- **Chapter 1: The Basics** — Primitive data types (`int`, `float`, `string`, `bool`), type inference (`:=`), mutability (`let mut`), arithmetic, and comments.
- **Chapter 2: Control Flow & Functions** — `if`/`elif`/`else`, `while` loops, function signatures (`fn name(param: type) -> ret:`), and recursion.
- **Chapter 3: Models & Object Orientation** — Data models (`model User: name: string`), constructors, instance methods, and model vectors.
- **Chapter 4: Memory Safety & Ownership** — Move semantics, reference borrowing (`&var`), stack versus heap allocation, and deterministic RAII cleanup.

### Part II: Concurrency, Asynchrony & Systems Programming
- **Chapter 5: Concurrency & Channels** — Lightweight thread spawning (`spawn task()`), channels (`channel[int]`), message passing, and thread synchronization.
- **Chapter 6: Async Programming & Metaprogramming** — Event loops, non-blocking I/O, compile-time macros, and syntax tree inspections.
- **Chapter 7: Database & Table Queries** — Native SQL/table data types, in-memory caching, indexing, and persistent key-value storage.
- **Chapter 8: Tensor Math & Hardware Peripherals** — SIMD vectors, matrix transformations, hardware buses (GPIO, I2C, SPI), and audio/video streaming.

### Part III: AI-First Architecture & Compiler Internals
- **Chapter 9: AI-First Compiler Features** — Self-documenting syntax models, tokenizer bridges, and vector embeddings.
- **Chapter 10: Custom Frontend & Backend File Extensions** — The `.cl` source file format, `.cld` documentation trees, and `.clb` binary bytecode bundles.
- **Chapter 11: Modular Web Development & High-Speed SSR** — Server-side rendering, component islands, and zero-JS templates.
- **Chapter 12-16: Advanced System Architecture** — Filesystem APIs, native networking, POSIX socket bindings, and binary serialization.
- **Chapter 17-24: Standard Modules & Algorithms** — Graph theory, dynamic programming, cryptographic primitives, and hashing algorithms.
- **Chapter 25-32: Production Deployment & Tooling** — LLVM compilation flags, cross-compilation (x86-64, ARM64), debugging, and package distribution.

---

## ⚡ Cluster Language Quick Syntax Reference

### 1. Variables & Models
```cluster
// Immutable assignment by default
x := 42
name := "Cluster"

// Mutable binding
let mut counter: int = 0
counter = counter + 1

// Structured data model
model ServerConfig:
    host: string
    port: int
    active: int
```

### 2. Functions & References
```cluster
fn process_buffer(cfg: ServerConfig, &buf: vector[string]) -> int:
    println("Processing on " + cfg.host + ":" + str(cfg.port))
    buf.push("item")
    return len(buf)
```

### 3. Concurrency
```cluster
fn worker(id: int):
    println("Worker " + str(id) + " running")

fn main():
    let mut i: int = 0
    while i < 4:
        spawn worker(i)
        i = i + 1
```

---

## 🤝 Contribution Guidelines & Zero-Dependency Rule

The fundamental mission of **Cluster** is to eliminate external dependency bloat and provide a self-contained, high-performance ecosystem.

### Core Rules for Contributions:
1. **100% Pure Cluster (`.cl`):** All new documentation code samples, std library references, and tutorials must be written purely in Cluster.
2. **Zero Foreign Dependencies:** Code examples must run without external third-party C/C++ libraries.
3. **Compiler-Only Execution:** Every snippet must compile and run using **only the Cluster compiler (`cluster`)**.
4. **Universal Device Portability:** Code must work consistently across all targets (x86-64, ARM64, embedded).

---

## 📜 License & Commercial Terms

Licensed under the **Cluster Community & Fair-Source License (CCFL v1.0 / MIT-Conditional)**:

- **100% Free for Independent & Research Use:**
  - Individual software developers, students, educators, academic researchers, non-profits, and scientific research groups have full rights to read, modify, and contribute without any license fees.
- **Commercial Enterprise License Required:**
  - Commercial companies, startups, and for-profit businesses using Cluster or its documentation in commercial publications or proprietary training require a commercial agreement.
  - Contact: **`zkalamgir@proton.me`** (`mohammadzaid/(cluster)`).

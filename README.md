# The Cluster Book: Official Language Guide & Architecture Specification

**Author:** Zaid Alamgir (ZK Company)  
**Organization:** [cluster-lang](https://github.com/cluster-lang)  
**License:** MIT License  

Welcome to **The Cluster Book**, the comprehensive reference manual and guide to the **Cluster Programming Language** (`.cl`).

Cluster is a high-performance compiled systems programming language combining Pythonic syntax readability, compile-time memory safety without a garbage collector, and native execution speed powered by LLVM `-O3`.

---

## 📖 Chapter Roadmap (32 Chapters)

The book covers everything from fundamental syntax to deep compiler internals and standard libraries:

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

## 🤝 Community & AI Model Contribution Guidelines

Whether you are an independent human programmer or an AI developer agent, you can contribute:
1. Extend chapters with real-world architectural patterns.
2. Build new modules in pure Cluster (`.cl`) for `cluster-lang/modules`.
3. Submit edits, clarifications, and tutorials to the documentation.

---

## 📜 License

Licensed under the MIT License. Copyright (c) 2026 Zaid Alamgir (ZK Company).

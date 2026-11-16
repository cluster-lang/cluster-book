---
layout: chapter
title: "Cluster-lang: Chapter 6 — Enums, Pattern Matching & Metaprogramming"
permalink: /chapter6/
prev_chapter: /chapter5/
prev_title: "Chapter 5 — Concurrency & Channels"
next_chapter: /chapter7/
next_title: "Chapter 7 — Database & Table Queries"
---

# Chapter 6: Enums, Pattern Matching & Metaprogramming — Expressive Type Safety & System Extensibility

Systems programming demands expressive types for state machines, protocol decoders, and command routers. Cluster combines first-class **Enums**, multi-branch **Pattern Matching (`match`)**, low-level **C/C++ Foreign Interop**, and compile-time **Metaprogramming**.

---

## 1. Enumerations (`enum`)

Enums define a closed set of symbolic named variants, compiled directly to compact integers in hardware registers:

```cluster
enum HttpMethod:
    GET
    POST
    PUT
    DELETE
    PATCH
    HEAD
    OPTIONS

enum ConnectionState:
    DISCONNECTED = 0
    CONNECTING   = 1
    CONNECTED    = 2
    DRAINING     = 3
    CLOSED       = 4
```

Enums eliminate "magic numbers" and ensure strict compile-time validation:
```cluster
model ClientSession:
    id: int
    state: ConnectionState
    method: HttpMethod
```

---

## 2. Multi-Branch Pattern Matching (`match`)

Instead of verbose `switch` statements or nested `if`/`elif` chains, Cluster provides structural pattern matching with the `match` keyword.

### Syntax & Case Branches
```cluster
fn handle_response_code(code: int) -> string:
    match code:
        case 200:
            return "OK"
        case 201:
            return "Created"
        case 400:
            return "Bad Request"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        default:
            return "Unhandled Status Code"
```

### Arrow Expression Syntax (`=>`)
For concise dispatchers, use the `=>` operator:
```cluster
fn route_cli_command(cmd: string):
    match cmd:
        "init"   => repo_init()
        "status" => repo_status()
        "commit" => repo_commit()
        "log"    => repo_log()
        _        => print_help()
```

The wildcard pattern `_` or `default:` acts as the exhaustive fallback case.

---

## 3. Low-Level Hardware Interoperability (`unsafe`, `extern`, `link`)

While Cluster is memory-safe by default, systems programming occasionally requires direct interaction with hardware registers, POSIX C ABIs, or kernel system calls.

### Foreign Function Interfaces (`extern fn`)
Declare external C functions with strict signatures:
```cluster
extern fn getaddrinfo(node: string, service: string, hints: int, res: int) -> int
extern fn freeaddrinfo(res: int) -> void
```

### Linking System Libraries (`link`)
Direct the native compiler backend to link system libraries without external build tools:
```cluster
link "pthread"
link "m"
link "ssl"
```

### Unsafe Blocks (`unsafe:`)
Explicitly scope regions that perform raw pointer manipulation or direct foreign memory access:
```cluster
fn read_hardware_port(port_addr: int) -> int:
    unsafe:
        // Perform direct hardware bus access or pointer offset
        raw_ptr := port_addr
        return *raw_ptr
```

### Inline Native Injection (`cpp_inject`)
For specialized optimizations or platform-specific inline assembly:
```cluster
fn fast_cpu_pause():
    cpp_inject "_mm_pause();"
```

---

## 4. Asynchronous Programming (`async` / `await`)

For cooperative event loops and high-throughput non-blocking I/O tasks:

```cluster
async fn fetch_telemetry(endpoint: string) -> string:
    // Non-blocking socket transfer
    data := await net_fetch_async(endpoint)
    return data

fn main():
    task := fetch_telemetry("https://api.internal/metrics")
    response := await task
    println("Received telemetry: " + response)
```

Under the hood, `async` routines compile to cooperative state machine fibers that yield CPU execution to the runtime worker pool without blocking operating system threads.

---

## 5. Architectural Comparison: Metaprogramming & Enums

| Feature | C / C++ | Rust | Go | Cluster |
| :--- | :--- | :--- | :--- | :--- |
| **Enum Type Safety** | ❌ Untyped integer tags | ✅ Algebraic Data Types | ❌ `iota` constants | **✅ Strict Enum Types** |
| **Pattern Matching** | ❌ `switch` (Integers only) | ✅ `match` | ❌ `switch` (basic) | **✅ Multi-Type `match` (`=>`)** |
| **Foreign C Interop** | Native | Complex `extern "C"` blocks | `cgo` (Massive overhead) | **⚡ Direct ABI Zero-Cost `extern`** |
| **Inline Injection** | `#pragma`, `asm` | `asm!` macro | ❌ None | **✅ `cpp_inject` & `unsafe:`** |

---

## 6. Summary

In this chapter, you learned:
- How to define symbolic types with `enum`.
- How to structure pattern-matching dispatchers with `match`, `case`, and `=>`.
- How to interface directly with OS libraries and hardware using `extern fn`, `link`, and `unsafe:`.
- How to leverage cooperative coroutines using `async` and `await`.

In the next chapter, we explore **Database & Table Queries** to build persistent storage engines and ORMs.

---
layout: chapter
title: "Cluster-lang: Chapter 2 — Control Flow & Functions"
permalink: /chapter2/
prev_chapter: /chapter1/
prev_title: "Chapter 1 — The Basics"
next_chapter: /chapter3/
next_title: "Chapter 3 — Models & Object Orientation"
---

# Chapter 2: Control Flow & Functions — Logic, Loops & Execution Models

Every high-performance system requires expressive control flow and modular functions. In Cluster, control flow is designed to be visually clean, eliminating superfluous parentheses and curly braces while preserving compile-time type verification.

---

## 1. Conditionals (`if`, `elif`, `else`)

Conditionals in Cluster evaluate boolean expressions. Parentheses around conditions are optional and discouraged for clean Pythonic readability.

### Syntax & Logical Operators
```cluster
status_code := 404

if status_code == 200:
    println("HTTP Request Succeeded")
elif status_code == 301 or status_code == 302:
    println("Redirect Encountered")
elif status_code >= 400 and status_code < 500:
    println("Client Error: " + str(status_code))
else:
    println("Server Error or Unknown State")
```

### Supported Boolean Operators:
- `and` — Logical conjunction (short-circuiting)
- `or` — Logical disjunction (short-circuiting)
- `not` — Logical negation
- `==`, `!=` — Equality and inequality
- `<`, `<=`, `>`, `>=` — Numeric comparisons

---

## 2. Iteration & Loops (`while`)

Cluster uses high-speed `while` loops as its core iteration primitive, which LLVM aggressively unrolls and vectorizes.

### Standard While Loop
```cluster
let mut i: int = 0
while i < 10:
    println("Iteration: " + str(i))
    i = i + 1
```

### Loop Control: `break` and `continue`
```cluster
let mut n: int = 0
while n < 20:
    n = n + 1
    
    // Skip even numbers
    if n % 2 == 0:
        continue
        
    // Terminate loop early
    if n > 15:
        break
        
    println("Odd value: " + str(n))
```

### Event Loops & Daemons (`while 1:`)
For persistent servers and background workers (such as the TCP accept loop in `cluster-redis`):
```cluster
while 1:
    client_fd := tcp_accept(server_fd)
    if client_fd < 0:
        break
    spawn handle_connection(client_fd)
```

---

## 3. Function Declarations

Functions are defined using the `fn` keyword. Parameters and return types are strictly typed for compile-time safety and optimal code generation.

### Standard Function
```cluster
fn calculate_hash(key: string, salt: int) -> int:
    let mut h: int = salt
    let mut i: int = 0
    let sz: int = len(key)
    while i < sz:
        h = ((h << 5) + h) + byte_at(key, i)
        i = i + 1
    return h
```

### Void Functions
When a function does not return a value, specify `-> void` or omit the return arrow:
```cluster
fn print_header(title: string) -> void:
    println("==================================================")
    println("  " + title)
    println("==================================================")
```

### Multiple Parameters & Default Patterns
```cluster
fn clamp(value: int, min_val: int, max_val: int) -> int:
    if value < min_val:
        return min_val
    if value > max_val:
        return max_val
    return value
```

---

## 4. Parameter Passing: Value vs. Reference (`&`)

Understanding how arguments are passed into functions is key to writing high-throughput systems:

### 1. Pass-by-Value (Default)
Primitive types (`int`, `float`, `bool`) and immutable variables are passed by value:
```cluster
fn increment_copy(val: int) -> int:
    return val + 1
```

### 2. Pass-by-Reference (`&`)
When modifying large structures (like a `vector[string]`) or appending output without allocating new memory, use the reference sigil `&`:
```cluster
fn collect_tokens(text: string, &out_tokens: vector[string]) -> void:
    let mut cur: string = ""
    let mut i: int = 0
    let sz: int = len(text)
    
    while i < sz:
        b := byte_at(text, i)
        if b == 32: // Space delimiter
            if len(cur) > 0:
                out_tokens.push(cur)
                cur = ""
        else:
            cur = cur + chr(b)
        i = i + 1
        
    if len(cur) > 0:
        out_tokens.push(cur)
```
> [!TIP]
> **Performance Impact:** Pass-by-reference avoids copying large vectors in heap memory, keeping function calls sub-nanosecond fast.

---

## 5. Recursion & Tail-Call Optimization

Cluster supports clean recursive functions. The compiler backend automatically optimizes tail-recursive calls into iterative jump instructions:

```cluster
fn factorial(n: int) -> int:
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

Recursive directory walking example (as seen in `cluster-rg`):
```cluster
fn walk_tree(dir: string, depth: int, max_depth: int, &out_files: vector[string]) -> void:
    if depth > max_depth:
        return
        
    items := fs_list_dir(dir)
    let mut i: int = 0
    while i < len(items):
        path := dir + "/" + items[i]
        if fs_is_dir(path) == 1:
            walk_tree(path, depth + 1, max_depth, out_files)
        else:
            out_files.push(path)
        i = i + 1
```

---

## 6. Comparison: Function Calling Across Languages

| Feature | Python | C++ | Rust | Cluster |
| :--- | :--- | :--- | :--- | :--- |
| **Call Overhead** | High (PyObject frame) | Zero (Native ABI) | Zero (Native ABI) | **⚡ Zero (Native LLVM inlined)** |
| **Type Checking** | Optional (Runtime hints) | Compile-time | Compile-time | **✅ Strict Compile-Time** |
| **Reference Syntax** | Invisible (Aliasing) | Complex (`const T&`, `T&&`) | Complex (`&`, `&mut`, lifetimes) | **✅ Clean `&` Reference Sigil** |
| **Tail-Call Elimination** | ❌ None (RecursionError) | Compiler-dependent | Limited | **✅ Automatic LLVM TCE** |

---

## 7. Summary

In this chapter, you learned:
- How to structure conditionals with `if`, `elif`, `else`, `and`, `or`, and `not`.
- How to write deterministic iteration loops with `while`, `break`, and `continue`.
- How to write typed function signatures using `fn name(...) -> type:`.
- How to use pass-by-reference (`&`) for zero-copy in-place modifications.

In the next chapter, we dive into **Models & Object Orientation** to encapsulate structured data and domain logic.

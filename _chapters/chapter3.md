---
layout: chapter
title: "Cluster-lang: Chapter 3 — Models & Object Orientation"
permalink: /chapter3/
prev_chapter: /chapter2/
prev_title: "Chapter 2 — Control Flow & Functions"
next_chapter: /chapter4/
next_title: "Chapter 4 — Memory Safety & Ownership"
---

# Chapter 3: Models & Object Orientation — Structured Data & Composition

Cluster rejects the bloated, fragile inheritance hierarchies of traditional object-oriented languages (like Java and C++). Instead, Cluster embraces **Data-Oriented Modeling** through the `model` keyword—giving you the simplicity of structs, the encapsulation of classes, and the memory performance of bare-metal C structures.

---

## 1. Defining Models

A model defines a structured data layout consisting of named, typed fields:

```cluster
model HttpRequest:
    method: string
    path: string
    protocol: string
    content_length: int
    keep_alive: int
```

Each field has an explicit type. Under the hood, LLVM compiles this model directly into a packed contiguous memory structure aligned to CPU cache boundaries.

---

## 2. Instantiating Models

Models can be constructed via standard positional arguments:

```cluster
req := HttpRequest("GET", "/api/v1/users", "HTTP/1.1", 0, 1)

// Field access using dot notation (.)
println("Method: " + req.method)
println("Path:   " + req.path)
```

### In-Place Field Mutation
When a model instance is held in a mutable binding, its fields can be updated directly:
```cluster
let mut session := HttpRequest("POST", "/login", "HTTP/1.1", 256, 1)
session.content_length = 512
session.path = "/auth"
```

---

## 3. Nested Models & Composition

Cluster favors composition over inheritance. Models can contain other models as fields:

```cluster
model UrlInfo:
    scheme: string
    host: string
    port: int

model ConnectionConfig:
    url: UrlInfo
    timeout_ms: int
    retry_count: int

// Instantiation with nested models
url := UrlInfo("https", "api.cluster-lang.org", 443)
cfg := ConnectionConfig(url, 5000, 3)

println("Target host: " + cfg.url.host + ":" + str(cfg.url.port))
```

---

## 4. Vectors of Models (`vector[Model]`)

In high-performance systems (such as the Git index in `cluster-git` or the hash table buckets in `cluster-redis`), you frequently store collections of models in a contiguous dynamic array:

```cluster
model TreeEntry:
    mode: string
    name: string
    sha1_hex: string

// Vector of models
let mut tree: vector[TreeEntry] = []

tree.push(TreeEntry("100644", "main.cl", "a1b2c3d4e5f6"))
tree.push(TreeEntry("100644", "README.md", "f6e5d4c3b2a1"))

// Iterate through model instances
let mut i: int = 0
let sz: int = len(tree)
while i < sz:
    let entry: TreeEntry = tree[i]
    println(entry.mode + " " + entry.name + " (" + entry.sha1_hex + ")")
    i = i + 1
```

---

## 5. Factory Functions & Constructor Patterns

To encapsulate validation, default values, and initialization logic, idiomatic Cluster code pairs models with factory functions:

```cluster
model RedisEntry:
    key: string
    val: string
    val_type: string
    expire_at: int
    access_count: int

// Factory constructor function
fn redis_entry_new_string(key: string, val: string, ttl_seconds: int) -> RedisEntry:
    let mut exp: int = -1
    if ttl_seconds > 0:
        exp = ttl_seconds
    return RedisEntry(key, val, "string", exp, 0)
```

Usage:
```cluster
entry := redis_entry_new_string("user:100", "Alice", 3600)
println("Stored key: " + entry.key + " with TTL: " + str(entry.expire_at))
```

---

## 6. Architectural Advantages: Models vs. Traditional OOP

| Metric | C++ Class / Struct | Java Class | Python Class | Cluster `model` |
| :--- | :--- | :--- | :--- | :--- |
| **Memory Layout** | Manual alignment | Heap pointer + header | Dict hashmap lookup | **⚡ Contiguous C-aligned structure** |
| **VTable Overhead** | 8 bytes per object | 8-16 bytes overhead | Dynamic type dict | **✅ 0 bytes (Zero vtable overhead)** |
| **Inheritance Bloat** | Fragile base classes | Monolithic hierarchies | Slow multiple inheritance | **✅ Clean, explicit composition** |
| **Field Lookup** | Fast (Offset) | Fast (Offset) | Very slow (Dict hash) | **⚡ Direct hardware memory offset** |
| **Cache Locality** | High | Low (Pointer chasing) | Extremely poor | **⚡ Optimal L1/L2 cache locality** |

---

## 7. Summary

In this chapter, you learned:
- How to define clean, strongly typed data structures using `model`.
- How to construct and mutate model instances.
- How to compose complex architectures using nested models.
- How to manage high-throughput collections using `vector[Model]`.
- Why Cluster's data-oriented model eliminates the memory and cognitive tax of legacy OOP.

In the next chapter, we will explore **Memory Safety & Ownership** to see how Cluster achieves rock-solid memory reliability without a garbage collector.

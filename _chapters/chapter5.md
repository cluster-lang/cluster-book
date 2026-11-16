---
layout: chapter
title: "Cluster-lang: Chapter 5 — Concurrency & Channels"
permalink: /chapter5/
prev_chapter: /chapter4/
prev_title: "Chapter 4 — Memory Safety & Ownership"
next_chapter: /chapter6/
next_title: "Chapter 6 — Async Programming & Metaprogramming"
---

# Chapter 5: Concurrency & Channels — True Parallelism Without a Global Lock

Modern servers have 16, 64, or 128 hardware CPU cores. Yet legacy languages fail to exploit this hardware cleanly:
- **Python** is shackled by the **Global Interpreter Lock (GIL)**, preventing multiple threads from executing CPU instructions simultaneously on separate cores.
- **Node.js** is single-threaded; a single heavy calculation freezes all incoming web requests on the event loop.
- **C/C++** offers raw threads, but lacks safety primitives, resulting in catastrophic race conditions, deadlocks, and corrupted heaps.

**Cluster delivers native concurrency:**  
Lightweight thread spawning via `spawn`, thread-safe channels (`Channel[T]`), and POSIX mutex synchronization—running with **true hardware parallelism and zero GIL**.

---

## 1. Spawning Concurrent Tasks (`spawn`)

In Cluster, launching an asynchronous task or concurrent fiber requires a single keyword: `spawn`.

### Spawning a Named Function
```cluster
fn process_chunk(chunk_id: int):
    println("Processing chunk #" + str(chunk_id) + " on worker thread")

fn main():
    let mut i: int = 0
    while i < 4:
        spawn process_chunk(i)
        i = i + 1
```

### Spawning Concurrent Network Handlers
In production (as seen in `cluster-redis`), `spawn` is used to service thousands of incoming client TCP connections concurrently without blocking the main listener:

```cluster
fn handle_client(client_fd: int) -> void:
    while 1:
        req := tcp_read(client_fd, 4096)
        if len(req) == 0:
            break
        // Process request and stream response
        tcp_write(client_fd, "+PONG\r\n")
    tcp_close(client_fd)

fn main():
    server_fd := tcp_listen(6379)
    println("Cluster server listening on port 6379...")
    
    while 1:
        client_fd := tcp_accept(server_fd)
        if client_fd >= 0:
            // Hand off client socket to concurrent worker
            spawn handle_client(client_fd)
```

---

## 2. Channels (`Channel[T]`)

Channels provide synchronized, race-free communication between concurrent tasks using the CSP (Communicating Sequential Processes) paradigm: *"Do not communicate by sharing memory; instead, share memory by communicating."*

### Creating a Channel
Declare a channel specifying the data type and queue capacity:
```cluster
ch := Channel[string](100)      // Buffered channel of strings with capacity 100
```

### Sending and Receiving
- `ch.send(data)` — Pushes an item into the channel queue (blocks if the channel buffer is full).
- `ch.recv()` — Pulls the next item from the channel (blocks until an item is available).

```cluster
fn producer(ch: Channel[int]):
    let mut i: int = 1
    while i <= 5:
        ch.send(i * 10)
        i = i + 1

fn consumer(ch: Channel[int]):
    let mut count: int = 0
    while count < 5:
        val := ch.recv()
        println("Received from channel: " + str(val))
        count = count + 1

fn main():
    data_channel := Channel[int](10)
    spawn producer(data_channel)
    consumer(data_channel)
```

---

## 3. Mutual Exclusion (`std/sync.cl`)

When multiple threads must mutate shared in-memory data structures (such as a database cache or an index table), Cluster provides low-overhead POSIX mutex locks:

```cluster
import "std/sync.cl"

model ThreadSafeCounter:
    count: int
    lock: int

fn increment(c: ThreadSafeCounter) -> void:
    mutex_lock(c.lock)
    c.count = c.count + 1
    mutex_unlock(c.lock)

fn main():
    m := mutex_create()
    counter := ThreadSafeCounter(0, m)
    
    // Spawn parallel workers updating the counter
    spawn increment(counter)
    spawn increment(counter)
```

---

## 4. Architectural Comparison: Concurrency Models

| Feature | Python (Threading) | Node.js | Go (Goroutines) | Cluster |
| :--- | :--- | :--- | :--- | :--- |
| **Multi-Core Parallelism** | ❌ Blocked by GIL | ❌ Single-threaded | ✅ Yes | **⚡ True Multi-Core Native Execution** |
| **GC Overhead** | Stop-the-world pauses | V8 GC cycles | Runtime GC pauses | **✅ 0 ms (Zero GC overhead)** |
| **Spawn Overhead** | OS thread overhead | Event queue callback | Runtime scheduler stack | **⚡ Lightweight POSIX thread pool** |
| **Syntax** | `threading.Thread(...)` | `async` / `await` callbacks | `go func()` | **✅ `spawn task()`** |
| **Type-Safe Channels** | ❌ `queue.Queue` (untyped) | ❌ None | ✅ `chan T` | **✅ `Channel[T]`** |

---

## 5. Summary

In this chapter, you learned:
- How to spawn concurrent tasks with the `spawn` keyword.
- How to build high-concurrency network servers servicing thousands of concurrent clients.
- How to coordinate concurrent tasks without locks using `Channel[T]`.
- How to protect critical shared state with `std/sync.cl` mutex locks.
- Why Cluster's GIL-free architecture extracts 100% of the computing power from modern multi-core CPUs.

In the next chapter, we will explore **Async Programming & Metaprogramming** for event loops and compile-time code transformations.

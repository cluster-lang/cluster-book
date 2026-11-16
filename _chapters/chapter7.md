---
layout: chapter
title: "Cluster-lang: Chapter 7 — Database & Table Queries"
permalink: /chapter7/
prev_chapter: /chapter6/
prev_title: "Chapter 6 — Enums, Pattern Matching & Metaprogramming"
next_chapter: /chapter8/
next_title: "Chapter 8 — Tensor Math & SIMD Acceleration"
---

# Chapter 7: Database & Table Queries — Relational Engines, Wire Drivers & Zero-GC Storage

In traditional programming languages, interacting with databases is notoriously painful:
- You must install external C driver libraries (like `libpq.so` or `libsqlite3.so`).
- Heavy ORMs (like Hibernate in Java, SQLAlchemy in Python, or Diesel in Rust) add thousands of lines of boilerplate, slow down builds, and introduce severe memory overhead.
- Periodic **Garbage Collector pauses** in Java, Go, and C# freeze database query execution, causing erratic latency spikes (high p99/p999 latency).

**Cluster solves this at the language level:**  
First-class model-driven table queries, built-in SQL execution, pure Cluster wire-protocol drivers, and **zero garbage collector pauses** ensuring sub-millisecond database response times.

---

## 1. Defining Schemas with Models & Creating Tables

In Cluster, database tables are defined using standard `model` structures. The compiler maps Cluster models directly to relational schemas without requiring manual SQL DDL migrations:

```cluster
import "cl-db"

model User:
    id: int
    username: string
    email: string
    balance: float
    is_active: bool

fn main():
    // Open or create a local database file (or ":memory:" for in-memory DB)
    db := DB("production.db")

    // Automatically generates table schema matching the User model
    db.create_table[User]()
```

Under the hood, the compiler creates the relational table with appropriate column types:
- `int` ➔ `INTEGER`
- `float` ➔ `REAL` / `DOUBLE PRECISION`
- `string` ➔ `TEXT` / `VARCHAR`
- `bool` ➔ `BOOLEAN` / `TINYINT`

---

## 2. Inserting & Mutating Records

Records are inserted directly using model instances:

```cluster
// Insert a new user record
user1 := User(1, "alice", "alice@cluster-lang.org", 1500.50, true)
db.insert(user1)

user2 := User(2, "bob", "bob@cluster-lang.org", 350.00, true)
db.insert(user2)
```

### Direct SQL Mutations (`db.execute`)
For bulk updates, complex condition evaluations, or deletions:
```cluster
// Parameterized SQL execution
db.execute("UPDATE User SET balance = balance + 100.0 WHERE is_active = 1")
db.execute("DELETE FROM User WHERE balance < 0.0")
```

---

## 3. Typed ORM-Style Queries (`db.query[Model]`)

Cluster provides type-safe, model-mapped query filtration:

```cluster
// Query records matching a boolean predicate
active_users := db.query[User]("balance > 500.0 AND is_active = 1")

let mut i: int = 0
let total: int = len(active_users)

while i < total:
    user := active_users[i]
    println("User #" + str(user.id) + ": " + user.username + " ($" + str(user.balance) + ")")
    i = i + 1
```

Benefits of typed model queries:
- **Zero Reflection Overhead:** Query results are unpacked directly into contiguous model vectors at native C speed.
- **Type Safety:** The returned vector is statically verified as `vector[User]`.

---

## 4. Pure Cluster Wire-Protocol Drivers (Zero C Dependencies)

Instead of relying on fragile foreign C/C++ dynamic libraries, the Cluster ecosystem provides native, zero-dependency wire drivers:

### 1. PostgreSQL Wire Protocol (`cl-postgres`)
Implemented in 100% pure Cluster, speaking the PostgreSQL 3.0 binary protocol directly over raw TCP sockets:
```cluster
import "cl-postgres"

fn main():
    client := pg_connect("127.0.0.1", 5432, "postgres", "secret", "mydb")
    res := client.query("SELECT id, name, salary FROM employees WHERE salary > $1", ["50000"])

    let mut i: int = 0
    while i < len(res.rows):
        row := res.rows[i]
        println("ID: " + row.values[0] + " | Name: " + row.values[1])
        i = i + 1
```

### 2. Redis Key-Value Store (`cl-redis` & `cluster-redis`)
Pure in-memory RESP protocol daemon and client handling over **204,000 requests/sec**:
```cluster
import "cl-redis"

fn main():
    client := redis_connect("127.0.0.1", 6379)
    client.set("cache:user:1", "Alice")
    val := client.get("cache:user:1")
    println("Cached value: " + val)
```

### 3. Vector & Search Databases (`cl-sqlite-vector`)
Embedded vector similarity indexing for AI embeddings and RAG pipelines:
```cluster
import "cl-sqlite-vector"

results := query_nearest("embeddings.db", "documents", query_vector)
```

---

## 5. In-Memory Fluent Query Pipelines (`cl-linq`)

When querying in-memory collections without a database, Cluster provides SQL-like fluent querying over `vector[T]`:

```cluster
import "cl-linq"

numbers: vector[int] = [12, 45, 68, 23, 89, 34, 90, 11]

total_count := count(numbers)           // 8
total_sum := sum(numbers)               // 372
top_three := take(numbers, 3)           // [12, 45, 68]
highest := max(numbers)                 // 90
lowest := min(numbers)                  // 11
```

---

## 6. Architectural Advantages: Database Systems in Cluster

| Feature | Java / Go / C# | C / C++ | Python | Cluster |
| :--- | :--- | :--- | :--- | :--- |
| **P99 Query Latency** | 10ms - 50ms (GC stalls) | Sub-millisecond | Slow (Interpreter) | **⚡ Sub-millisecond (0 ms GC pause)** |
| **Memory Corruption** | Safe (Managed) | ❌ High risk (Segfaults) | Safe | **✅ 100% Memory-Safe by Default** |
| **Driver Dependencies** | Heavy JDBC / Go modules | Complex `.so` / `.dll` | Heavy C-extensions | **✅ Pure Cluster Wire Drivers** |
| **Binary Output** | Heavy JAR / VM | Standalone | Requires CPython | **✅ Single Standalone ELF Executable** |

---

## 7. Summary

In this chapter, you learned:
- How to define database tables from models using `db.create_table[Model]()`.
- How to perform type-safe ORM queries with `db.query[Model]`.
- How to execute direct parameterized SQL with `db.execute()`.
- How pure wire drivers (`cl-postgres`, `cl-redis`) eliminate external C client dependencies.
- How in-memory fluent queries (`cl-linq`) bring SQL-like power to local data vectors.

In the next chapter, we dive into **Tensor Math & SIMD Acceleration** to see how Cluster executes machine learning and numerical computing at hardware limits.

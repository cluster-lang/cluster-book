---
layout: chapter
title: "Chapter 36: Error Handling, Result Monads & Robust Panics (std/result, Result[T, E], assert)"
permalink: /chapter36/
prev_chapter: /chapter35/
prev_title: "Chapter 35"
---

# Chapter 36: Error Handling, Result Monads & Robust Panics

Software reliability depends on predictable, explicit error handling. Legacy languages force developers between two problematic extremes:
* **The C Approach:** Returning integers (`-1`, `NULL`) that developers frequently forget to check, leading to buffer overflows and null pointer dereferences.
* **The C++/Java Approach:** Invisible runtime exceptions (`throw`/`catch`) that add hidden control flow, bloat binaries with multi-megabyte stack-unwinding tables, and degrade performance.

Cluster rejects both approaches. Instead, it unifies **zero-cost algebraic `Result[T, E]` monads**, **expressive syntactic operators (`?:`, `??`, `??=`, `?.`)**, and **fail-closed assertion contracts (`assert`)**.

This chapter covers:
1. **The `std/result.cl` Standard Library Module** (`ok`, `err`, `unwrap`, `unwrap_or`).
2. **Syntactic Null-Coalescing & Fallback Operators** (`?:`, `??`, `??=`, `?.`).
3. **Pattern Matching on Results** with `match` and algebraic variants.
4. **Fail-Closed Assertion Contracts & Systems Panics** (`assert`).
5. **Production Case Study**: Robust configuration parser with zero exception overhead.

---

## 1. The Result Monad (`std/result.cl`)

In Cluster, functions that can fail return an explicit `Result` type rather than throwing an exception or returning magic error numbers.

### Functions and Signatures:
```python
import "std/result.cl"

fn divide(numerator: int, denominator: int) -> Result:
    if denominator == 0:
        return err_with_code("Division by zero is undefined", 400)
    return ok(numerator / denominator)

fn main() -> int:
    res1 := divide(100, 5)
    if is_ok(res1):
        println("Result: " + to_string(unwrap(res1)))   // 20

    res2 := divide(100, 0)
    if is_err(res2):
        println("Error [" + to_string(err_code(res2)) + "]: " + unwrap_err(res2))
        
    // Safe fallback with unwrap_or
    safe_val := unwrap_or(res2, -1)
    println("Fallback value: " + to_string(safe_val))    // -1
    return 0
```

---

## 2. Syntactic Ergonomics for Fallbacks & Null Safety

Cluster provides compiler-level syntactic sugar to handle potential errors and absent values in a single line without multi-level `if` nesting:

### A. The Elvis Operator (`?:`):
Evaluates the left expression; if falsy (zero, empty string, or null), returns the right expression:
```python
host := getenv("HOST") ?: "127.0.0.1"
port := get_custom_port() ?: 8080
```

### B. The Null-Coalescing Operator (`??`):
Returns the left operand if non-null, or evaluates the right fallback:
```python
user_alias := profile.nickname ?? profile.legal_name
```

### C. Null-Coalescing Assignment (`??=`):
Assigns a value to a variable only if the variable is currently null or empty:
```python
mut session_cache := get_cached_token()
session_cache ??= request_new_token_from_auth()
```

### D. Safe Navigation / Optional Chaining (`?.`):
Safely accesses nested members, returning `null` if any intermediate reference is nil:
```python
email := user?.organization?.contact?.email
```

---

## 3. Pattern Matching on Results

When a function returns different error codes or payloads, `match` guarantees clean, branchless handling:

```python
enum AuthResult:
    Success(token: string)
    InvalidCredentials(attempts_left: int)
    AccountLocked(unlock_time: int)

fn authenticate_user(user: string, pass: string) -> AuthResult:
    if user == "admin" and pass == "secret":
        return AuthResult::Success(token="cl_jwt_98412")
    return AuthResult::InvalidCredentials(attempts_left=2)

fn main() -> int:
    result := authenticate_user("admin", "wrong_pass")
    
    match result:
        case AuthResult::Success(s):
            println("Authenticated! Token: " + s.token)
        case AuthResult::InvalidCredentials(ic):
            println("Access denied. Attempts left: " + to_string(ic.attempts_left))
        case AuthResult::AccountLocked(al):
            println("Account locked. Unlock in: " + to_string(al.unlock_time) + "s")
    
    return 0
```

---

## 4. Fail-Closed Invariant Contracts (`assert`)

For programming invariants that should never occur in valid code, Cluster uses fail-closed `assert(condition)`.

```python
fn process_transaction(amount: float, balance: float) -> float:
    // Enforce invariant contract: amount must always be positive
    assert(amount > 0.0)
    assert(balance >= amount)
    
    return balance - amount
```

### Safety Guarantees:
* **Deterministic Termination:** If an assertion fails, the Cluster runtime prints the exact file, line number, and expression, then terminates execution immediately.
* **Zero Corrupted State:** Unlike languages that allow memory corruption or state desynchronization after a soft error, fail-closed contracts isolate faults before data can be persisted.

---

## 5. Complete Production Case Study: Configuration Parser

The following pure Cluster application demonstrates loading, parsing, and validating a service configuration with full zero-cost error propagation:

```python
import "std/result.cl"
import "std/crypto.cl"

model Config:
    host: string
    port: int
    secret_hash: string

fn parse_config(content: string) -> Result:
    if len(content) == 0:
        return err_with_code("Empty configuration provided", 101)
        
    // Validate required host
    let mut host := "127.0.0.1"
    let mut port := 8080
    
    // Compute cryptographic integrity hash
    hash := sha1(content)
    
    return ok(1) // Configuration parsed successfully
```

By unifying explicit Result types, algebraic pattern matching, syntactic fallback operators, and fail-closed assertion contracts, Cluster allows developers to build rock-solid, production-grade systems with **zero runtime exception overhead** and **zero memory leaks**.

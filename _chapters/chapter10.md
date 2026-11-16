---
layout: chapter
title: "Cluster-lang: Chapter 10 — Custom Frontend & Backend File Extensions"
permalink: /chapter10/
prev_chapter: /chapter9/
prev_title: "Chapter 9 — AI-First Compiler Features & Native Model Execution"
next_chapter: /chapter11/
next_title: "Chapter 11 — Modular Web Development & High-Speed SSR"
---

# Chapter 10: Custom Frontend & Backend File Extensions — The Domain-Specific Asset Pipeline

Cluster is not just a language; it is a unified application fabric. To eliminate the messy sprawl of YAML, JSON, Markdown, HTML template engines, and separate build preprocessors, Cluster introduces first-class **domain-specific file extensions** that compile directly into native machine code.

---

## 1. `.cld` & `.clb` (Cluster Documents & High-Density Binary Bundles)

As introduced in `cluster-ssg`, the `.cld` (Cluster Document) format replaces legacy Markdown and HTML with a structured, high-speed document AST, while `.clb` provides bit-for-bit binary bytecode serialization.

### Cluster Document (`article.cld`)
```cld
---
title: "Zero-Dependency Architecture"
author: "mohammadzaid/(cluster)"
date: "2026-11-16"
layout: "post"
---

# Zero-Dependency Architecture

Modern systems programming requires eliminating third-party bloat.

<island name="TelemetryWidget" client:load refresh_interval="5" />
```

### Compiling and Unpacking Binary Bundles
```bash
# Compile human-readable .cld document into high-density binary bundle (.clb)
cluster-ssg pack article.cld article.clb

# Decompile .clb bytecode back into clean .cld source
cluster-ssg unpack article.clb reconstructed.cld
```
In browser applications, `.clb` bundles are streamed across the wire for default source-code obfuscation and binary payload security.

---

## 2. `.clcfg` (Cluster Static Configurations)

The `.clcfg` extension allows writing app configurations in a clean key-value format. The compiler transpiles these configs into compile-time constants, eliminating runtime config parsing and startup delays:

### Configuration File (`config.clcfg`)
```ini
app_name = "Cluster App"
port = 8080
debug = true
rate_limit = 1.5
```

### Usage in Code (`main.cl`)
```cluster
import "config.clcfg" as config

fn main():
    println(config.app_name)
    println("Port: " + str(config.port))
```

---

## 3. `.clx` (Cluster UI Templates)

The `.clx` extension supports writing HTML/XML-like UI components with embedded control flows. They compile to native string-builder operations, making server-side HTML rendering (SSR) hundreds of times faster than Node.js.

### UI Component File (`TodoItem.clx`)
```html
<component name="TodoItem" item: string, done: bool>
    <div class="item">
        <span>{item}</span>
        {if done}
            <span class="badge">Completed</span>
        {else}
            <span class="badge">Pending</span>
        {end}
    </div>
</component>
```

### Usage in Code (`main.cl`)
```cluster
import "TodoItem.clx" as ui

fn main():
    html := ui.TodoItem("Write native compiler", true)
    println(html)
```

---

## 4. `.clq` (Cluster Pre-compiled Queries)

The `.clq` extension allows writing database queries mapped to functions. The compiler transpiles these into pre-prepared database bindings, eliminating runtime SQL parsing latency.

### Database Query File (`queries.clq`)
```sql
database: "app.db"

query get_user_by_id(id: int):
    SELECT id, name, email FROM users WHERE id = ?1;
```

### Usage in Code (`main.cl`)
```cluster
import "queries.clq" as q

fn main():
    results := q.get_user_by_id(42)
    println("Records found: " + str(len(results)))
```

---

## 5. `.clp` (Cluster Pattern Parser)

The `.clp` extension enables declaring pattern parsers for structured logs, CSVs, or key-value data. The compiler transpiles these pattern rules into optimized, zero-dependency parsing models.

### Parser Definition File (`UserCSV.clp`)
```text
parser UserCSV:
    format: "{id:int},{name:string},{email:string}"
```

### Usage in Code (`main.cl`)
```cluster
import "UserCSV.clp" as parser

fn main():
    csv_line := "101,Bob,bob@example.com"
    user := parser::parse_UserCSV(csv_line)
    
    println("User ID: " + str(user.id))
    println("Name:    " + user.name)
    println("Email:   " + user.email)
```

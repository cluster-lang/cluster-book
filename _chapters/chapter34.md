---
layout: chapter
title: "Chapter 34: Standard Document, Data & Physics Wire Formats (std/cld, std/clb, std/cls, std/clw)"
permalink: /chapter34/
prev_chapter: /chapter33/
prev_title: "Chapter 33"
next_chapter: /chapter35/
next_title: "Chapter 35"
---

# Chapter 34: Standard Document, Data & Physics Wire Formats

To eliminate third-party JSON parsers, heavy Protocol Buffer compilers, slow XML parsers, and external physics middleware, the Cluster language provides a built-in suite of domain-specific data and wire formats directly in its standard library (`std/`).

This chapter covers:
1. **Cluster Document Engine (`std/cld.cl`)**: Markdown/HTML parser, AST generation, and Astro-style islands.
2. **Cluster Binary Bundle (`std/clb.cl`)**: Ultra-fast, zero-reflection data interchange replacing JSON and Protobuf.
3. **Cluster Schema & Contract Validation (`std/cls.cl`)**: Declarative type enforcement, range bounds, and runtime assertions.
4. **Cluster Spatial Physics Grid (`std/clw.cl`)**: 3D voxel chunks, rigidbodies, friction, raycasting, and AABB collision.

---

## 1. Cluster Document Engine (`std/cld.cl`)

The `.cld` format provides a modern static site and documentation document model that combines the simplicity of Jekyll/Markdown frontmatter, the compilation speed of Hugo, and the partial hydration island architecture of Astro.

### Core Data Models:
```python
model CldMeta:
    title: string
    author: string
    date: string
    layout: string
    tags: string
    slug: string
    description: string

model CldBlock:
    kind: string     // "h1", "h2", "p", "code", "quote", "list", "island", "table"
    content: string
    extra: string    // Language for code blocks, heading level

model CldIsland:
    name: string
    directive: string // "load", "idle", "visible", "zero-js"
    props: string
    fallback: string

model CldDoc:
    meta: CldMeta
    blocks: vector[CldBlock]
```

### Parsing and Rendering Documents:
```python
import "std/cld.cl"

fn main() -> int:
    sample := "title: Getting Started with Cluster\n" +
              "author: mohammadzaid\n" +
              "layout: default\n" +
              "===\n" +
              "# Welcome to Cluster\n\n" +
              "Cluster delivers **zero-dependency** native execution.\n\n" +
              "<island name=\"Counter\" client:visible count=\"10\" />"

    // Parse into high-performance in-memory AST
    doc := cld_parse_text(sample)
    
    println("Document Title: " + doc.meta.title)
    println("Author:         " + doc.meta.author)
    println("Total Blocks:   " + to_string(len(doc.blocks)))

    // Render directly into HTML layout template
    layout := "<html><head><title>{{ title }}</title></head><body>{{ content }}</body></html>"
    html := cld_render_page(doc, layout)
    println("Rendered HTML:\n" + html)
    return 0
```

---

## 2. Cluster Binary Bundle (`std/clb.cl`)

Traditional formats like JSON are human-readable but computationally slow: they require text scanning, string allocations, and float conversions. Conversely, Protocol Buffers require external `protoc` compilers and brittle build-phase code generation.

The Cluster Binary Bundle (`.clb`) provides typed key-value serialization that reads and writes binary records with sub-microsecond roundtrip speed and **zero external dependencies**.

### Working with Binary Bundles:
```python
import "std/clb.cl"

fn main() -> int:
    // Create a new binary bundle document
    doc := clb_new()
    
    // Store typed fields
    clb_set_string(doc, "username", "alice")
    clb_set_int(doc, "user_id", 10042)
    clb_set_float(doc, "account_balance", 2540.50)
    clb_set_bool(doc, "is_verified", 1)

    // Save directly to binary disk file (atomic, compact)
    clb_save("profile.clb", doc)
    println("Binary bundle saved to profile.clb")

    // Load instantly from disk
    loaded := clb_load("profile.clb")
    name := clb_get_string(loaded, "username", "anonymous")
    uid := clb_get_int(loaded, "user_id", 0)
    bal := clb_get_float(loaded, "account_balance", 0.0)

    println("User: " + name + " (ID: " + to_string(uid) + "), Balance: " + to_string(bal))
    return 0
```

---

## 3. Cluster Schema & Contract Validation (`std/cls.cl`)

To guarantee data integrity across microservices, REST APIs, or local configuration files without importing massive schema validation libraries, Cluster introduces declarative schemas (`.cls`):

```python
import "std/cls.cl"
import "std/clb.cl"

fn main() -> int:
    // Define API payload contract
    schema := cls_new("UserProfileSchema")
    
    // Field constraints: name, type, required, min, max
    cls_add_string(schema, "username", 1, 3, 32, "")
    cls_add_int(schema, "age", 1, 18, 120)
    
    status_roles: vector[string] = ["admin", "developer", "guest"]
    cls_add_enum(schema, "role", 1, status_roles)

    // Validate bundle document against contract
    doc := clb_new()
    clb_set_string(doc, "username", "zaid")
    clb_set_int(doc, "age", 24)
    clb_set_string(doc, "role", "developer")

    if cls_validate(schema, doc) == 1:
        println("✓ Contract verification passed: Payload conforms to schema.")
    else:
        println("✗ Contract violation:\n" + cls_get_errors(schema))

    return 0
```

---

## 4. 3D Spatial Physics Grid (`std/clw.cl`)

Cluster includes a dedicated 3D spatial voxel, terrain, and physics chunking engine (`std/clw.cl`). It is designed for voxel games, autonomous robotics simulation, and spatial raycasting.

### Core Architecture:
* **Voxel Chunking:** World coordinates are partitioned into $16 \times 16 \times 16$ or $32 \times 32 \times 32$ spatial chunks, allocated on-demand in memory.
* **Material Properties:** Materials define `solid`, `friction`, `restitution` (bounciness), and `density`.
* **Spatial Collision:** Axis-Aligned Bounding Box (AABB) box checks and DDA fast voxel raycasting are implemented in pure Cluster.

```python
import "std/clw.cl"

fn main() -> int:
    // Initialize world: name, cell size (1.0m), chunk size (16), gravity (-9.81 m/s^2)
    world := clw_world_new("SimulationSector1", 1.0, 16, -9.81)

    // Register physical materials
    // id, name, solid, friction, restitution, density
    clw_add_material(world, 1, "Stone", 1, 0.6, 0.1, 2600.0)
    clw_add_material(world, 2, "Water", 0, 0.1, 0.0, 1000.0)

    // Place voxel blocks in 3D world space
    clw_set_block(world, 0, 0, 0, 1)
    clw_set_block(world, 0, 1, 0, 1)
    clw_set_block(world, 0, 2, 0, 2)

    // Perform AABB Box Collision Check
    has_collision := clw_check_box_collision(world, -0.5, -0.5, -0.5, 0.5, 0.5, 0.5)
    if has_collision == 1:
        println("Rigid body collision detected at origin!")

    // Fast DDA Voxel Raycast (origin -> direction -> max_dist)
    hit_block := clw_raycast(world, 0.0, 10.0, 0.0, 0.0, -1.0, 0.0, 20.0)
    println("Raycast hit material ID: " + to_string(hit_block))

    // Serialize world directly to binary disk archive
    clw_save("world.clw", world)
    println("World state saved to world.clw.")
    return 0
```

By providing these standard wire modules directly within the compiler toolchain, developers can construct entire static site engines, microservices, and 3D simulation tools with **zero external dependencies**.

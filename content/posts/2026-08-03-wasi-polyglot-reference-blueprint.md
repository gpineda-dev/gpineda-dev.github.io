---
title: "The WASI V2 Polyglot Blueprint: Executing Pure Go in Python and JS Without Rewriting"
date: 2026-08-03T20:00:00+02:00
draft: true
summary: "How I compile pure Go algorithmic primitives from Ouvrage Systems (ouvrage-kern-go) into WASI Preview 2 components to consume them natively and type-safely in Python and Node.js."
tags: ["webassembly", "wasi", "golang", "python", "architecture"]
showToc: true
math: true
mermaid: true
---

## 1. The Philosophy of Ouvrage-Kern

When I designed the **`ouvrage-kern-go`** (`okern`) core library, my goal was straightforward: collect my core algorithms (formal language theory, Pratt parsers, tree data structures, and applied mathematics) into a single, reusable Go module.

To maintain clarity and modularity across the repository, I established four simple engineering rules (documented in ADR-000):
1. **Strict Separation of Concerns**: Physical package isolation to prevent circular imports.
2. **Generic Abstractions**: Leveraging Go generics (my `/cs/ds/tree` package visualizes both ASTs and directory structures without modification).
3. **Zero-Dependency Core**: The core engine carries zero external dependencies. Executable CLI tooling (`cmd/oklang`) is isolated at the boundary.
4. **Continuous Compliance**: Automated build-time tests walk and validate the import graph on every commit.

---

## 2. The Dilemma: Rewriting in Python or Compiling to WASI?

While I rely on Go for low-level systems engineering, auxiliary components in the Ouvrage ecosystem (ingestion tools, semantic analysis scripts, IDE extensions) need to run in **Python** or **TypeScript/Node.js**.

Facing this, I had three options:
* **Option A (Manual Rewrite)**: Rewrite Pratt parsers and AST structures by hand in Python and JS. However, maintaining complex parser logic across multiple languages introduces an inevitable risk of semantic drift and duplication.
* **Option A' (CLI Code Generation)**: Add a code generation command to the Ouvrage CLI (`oklang codegen -g grammar.yml --target py`) to emit a zero-abstraction `parser.py` tailored to a specific grammar. A clean evolution path for `oklang`, but not the immediate priority for the `okern` core engine.
* **Option B (WASI V2 - Selected R&D)**: Compile the Go engine into a deterministic WebAssembly component and generate typed native wrappers via the Component Model.

I chose to push **Option B** to validate polyglot distribution. Last weekend, I published a reference laboratory: [**`labs-wasi-polyglot-bindings-reference`**](https://ouvrage-systems.github.io/labs-wasi-polyglot-bindings-reference/) to get hands-on experience with the transition from WASM 0.1 $\rightarrow$ WASI Preview 1 $\rightarrow$ WASI Preview 2 (Component Model).

Rather than hiding behind automated `make build-v2` scripts, this post walks through a hands-on exercise: **adding a new method (listing keys) to our Go store and executing the WASI compilation and inspection steps manually.**

---

## 3. Why the WASI Preview 2 Component Model?

Rather than using legacy WASM 0.1 (restricted to browsers and un-typed linear memory), I rely on **WASI Preview 2 (WASI 0.2)** and its *Component Model*.

The key advantage? We declare strict interfaces using **WIT** (*WebAssembly Interface Type*) files, and the WASM binary cleanly exports stateful objects with constructors and methods inside isolated heap memory.

---

## 4. Step-by-Step Case Study: Adding `ListKeys` and Dissecting the Component

For this exercise, we start from a simple Go in-memory `KVStore` (`pkg/store/store.go`). We will add the `ListKeys() []string` method and update the entire pipeline without relying on build script automation.

### 4.1 The WIT Interface (The Binary Contract)
Before compiling, we declare the component contract inside our `store.wit` file:

```wit
package ouvrage:lab-wasi-demo;

interface store {
    resource kv-store {
        constructor();
        set: func(key: string, value: string);
        get: func(key: string) -> option<string>;
        delete: func(key: string) -> bool;
        list-keys: func() -> list<string>;
    }
}

world store-service {
    export store;
}
```

### 4.2 The Memory Model: Linear Heap & Opaque Handles
How does the Python host safely interact with a Go struct without risking memory corruption?

1. **Heap Memory Isolation**: The Go instance's linear WASM memory is strictly sandboxed from the host Python process.
2. **The Handle Mechanism**: The Go constructor `[constructor]kv-store` allocates the struct in WASM memory and returns an opaque integer identifier (`handle`).
3. **Context Passing**: Subsequent method calls (`set`, `get`, `delete`) pass this `handle` back to the WASM engine to target the correct in-memory struct instance.

```text
[ Host Python ] ──( Calls set(handle=1, "key", "val") )──► [ WASM Sandbox / Go Heap ]
                                                             └── Pointer #1 (KVStore)
```

### 4.3 Dissecting the Component (`wasm-tools`)
Using `wasm-tools`, we can directly inspect and verify the compiled `.wasm` component interface generated by `tinygo`:

```bash
# Inspecting the WIT interface of the compiled WASM component
wasm-tools component wit build/store.wasm
```

---

## 5. Autonomous Execution: Two Memory Isolation Strategies

In WASI Preview 2, systems engineers can choose between two memory isolation strategies depending on security requirements and memory budget:

```mermaid
graph TD
    subgraph Strategy1 ["Strategy A: Distinct Sandboxes (Absolute Hardware Isolation)"]
        Proc1["main.py"]
        Store1["wasmtime.Store #1"] ──► Heap1["Go Heap #1 (Alpha)"]
        Store2["wasmtime.Store #2"] ──► Heap2["Go Heap #2 (Beta)"]
        Proc1 --> Store1
        Proc1 --> Store2
    end

    subgraph Strategy2 ["Strategy B: Shared Handles (Maximum Density & Zero Overhead)"]
        Proc2["main.py"]
        SingleStore["Single wasmtime.Store"]
        HeapShared["Single Go Heap"]
        Handle1["Handle #1 (Alpha)"]
        Handle2["Handle #2 (Beta)"]
        
        Proc2 --> SingleStore
        SingleStore --> HeapShared
        HeapShared --> Handle1
        HeapShared --> Handle2
    end
```

### Strategy A: Multi-Stores (Physical Sandboxing)
Each Go instance lives in its own dedicated `wasmtime.Store()`. If instance Alpha experiences heap corruption or a panic, instance Beta remains physically protected in host RAM and continues executing normally:
```python
# Instantiating two distinct WASM sandboxes
store_alpha = wasmtime.Store(engine)
instance_alpha = linker.instantiate(store_alpha, component)

store_beta = wasmtime.Store(engine)
instance_beta = linker.instantiate(store_beta, component)
```

### Strategy B: Single-Store & Handles (High Density & Zero Overhead)
A single `wasmtime.Store()` is allocated (Singleton pattern). Calling the Go constructor returns opaque integer `handles`. This strategy is ideal for managing thousands of lightweight domain objects inside the same host process without multiplying runtime overhead:
```python
# A single shared memory sandbox
store = wasmtime.Store(engine)
instance = linker.instantiate(store, component)

# Allocating two distinct KVStores via Handles
handle_alpha = ctor_func(store) # Handle 1
handle_beta = ctor_func(store)  # Handle 2

set_func(store, handle_alpha, "env", "production")
set_func(store, handle_beta, "env", "staging")
```

---

## 6. Observability & Waterfall Tracing (JSON Traces & Streamlit)

To accurately measure component instantiation and handle lifecycles in memory, we can instrument execution to emit a trace file in **Chrome Tracing / Perfetto** format (`trace.json`).

### 6.1 The Chrome Tracing Dumper (`tracer.py`)
```python
# TODO @gpineda: Fill in WasiTracer skeleton with Perfetto / Chrome Tracing format
class WasiTracer:
    def __init__(self, output_file="trace.json"):
        self.events = []
        self.output_file = output_file
        
    def log_event(self, name: str, cat: str, ph: str, ts_ns: int, args: dict = None):
        pass

    def save(self):
        pass
```

### 6.2 Waterfall Visualization (Streamlit Dashboard)
```python
# TODO @gpineda: Fill in Streamlit app.py script for rendering the Waterfall chart (Plotly/Altair)
import streamlit as st
import json

st.title("🔬 WASI Execution Trace & Waterfall Viewer")
# Insert Waterfall rendering logic from trace.json
```

---

## 7. Key Takeaways

This WASI Preview 2 component approach validates three core engineering principles:
1. **Zero Code Duplication**: Go remains the single source of truth for core algorithms.
2. **Type-Safety & Isolation**: Python interacts with WIT-typed APIs without exposing host system memory to corruption.
3. **Minimal Overhead**: The compiled WASM component stays under a few megabytes and boots in microseconds.

The reference laboratory [**`labs-wasi-polyglot-bindings-reference`**](https://ouvrage-systems.github.io/labs-wasi-polyglot-bindings-reference/) serves as the foundational packaging blueprint across the Ouvrage Systems constellation.

---

> 🛠️ *This post was co-designed with Gemini (Antigravity) using our [Augmented Pair-Authoring Workflow](/posts/2026-08-04-augmented-pair-authoring-workflow) (see also our [Exoskeleton Manifesto](/posts/2026-08-04-exoskeleton-manifesto)).*



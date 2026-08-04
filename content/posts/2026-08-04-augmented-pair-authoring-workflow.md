---
title: "From Raw Notes to Published Post: My Augmented Pair-Authoring Workflow"
date: 2026-08-04T03:15:00+02:00
draft: false
summary: "A practical breakdown of how I collaborate with LLMs to translate raw engineering notes into structured, bilingual articles without losing human voice or technical accuracy."
tags: ["ai", "meta", "writing", "workflow"]
showToc: true
---

## 1. Why Separate Code Generation from Technical Writing?

In our [Exoskeleton Manifesto](/posts/2026-08-04-exoskeleton-manifesto), we outlined the strict separation between human architecture and machine implementation. However, applying AI to **technical writing** presents a unique challenge compared to code generation:

* **In Code Generation**: The output is validated by a compiler, unit tests, and runtime metrics. Correctness is binary.
* **In Prose Generation**: The output must preserve human authenticity, personal history, and nuanced engineering decisions. Naive LLM prose leads to generic, buzzword-heavy marketing fluff.

To prevent this, I use a structured **Pair-Authoring Workflow** where the AI acts strictly as an editor and translator, while I remain the sole source of technical truth and personal narrative.

---

## 2. The Division of Labor

```text
[ Guillaume (@gpineda) ]                                 [ AI Exoskeleton ]
───────────────────────                                 ──────────────────
Raw thoughts & anecdotes  ─────( 1. Raw Input )────►   Structure & Categorization
Technical constraints      ◄───( 2. Clarification )───   Formatting & Markdown
Final Veto & Fact Check   ─────( 3. Validation )─────►   Bilingual Translation
```

1. **The Human Source**: Provides raw notes, technical decisions, historical anecdotes, and hard constraints.
2. **The Exoskeleton**: Organizes bullet points, suggests progressive disclosure layouts, highlights logical gaps, and handles bilingual Markdown translation (`en` / `fr`).
3. **The Human Veto**: Reviews every line, strips LLM hyperbole, and ensures 100% alignment with real-world facts.

---

## 3. Step-by-Step Case Study: From System Requirement to WASM Architecture

To illustrate this workflow, here is a trace from a real session translating a complex engineering requirement (designing the `ocalque` AOT templating engine and WASM compilation of `ouvrage-kern-go`) into a cohesive architectural breakdown.

### Step 1: Raw Human Input (`@gpineda`)
A stream of dense technical thoughts detailing tooling constraints and compiler goals:
> *"For my legacy C codebase in Visual Studio Express, I don't want a text-replacement engine like Jinja that corrupts source files. I need `ocalque`, an AOT templating engine using the Shadow Pattern (directives embedded in native comments) so the code stays pristine and compilable in dev. The core multi-grammar Pratt parser (`pkg/lang` inside `ouvrage-kern-go`) is pure Go with zero dependencies. The `oklang` CLI orchestrates it, but the `pkg/lang` package needs to be compiled to WASI Preview 2 to embed this Pratt engine (AST building, evaluation, Mermaid/DOT exports) natively in Python and JS without rewriting the parser. Hence the `wasi-polyglot-bindings-reference` lab opened this week."*

### Step 2: AI Exoskeleton Reframing & Structuring
The AI identifies responsibility boundaries and suggests isolating the layers for the reader:
> *"The architectural intent is extremely clear. Let's strictly decouple the CLI (`oklang`) from the core library (`pkg/lang`). We isolate three layers: 1) The product requirement (Shadow Pattern for pristine dev C code), 2) The theoretical core (Multi-grammar Pratt engine built for ASTs, evaluation, and Mermaid/DOT exports), and 3) Polyglot deployment (compiling `pkg/lang` to WASM/WASI Preview 2 for Python and Node.js). Let's structure this logically."*

### Step 3: Final Reviewed Technical Output
The final architectural paragraph, drafted and verified by the engineer:
> *“To evaluate directives for the `ocalque` AOT templating engine without polluting original C code in historic IDEs (the Shadow Pattern), we isolated its parsing engine into `ouvrage-kern-go` (`pkg/lang`). Written in pure Go with zero dependencies, this package implements a multi-grammar Pratt parser capable of constructing ASTs, evaluating expressions, and exporting graph representations (Mermaid, DOT, GraphML). To reuse this algorithmic core across other ecosystems (Python tooling, TypeScript LSP servers) without rewriting the parser, `pkg/lang` is compiled into a WebAssembly component (WASI Preview 2)—a pattern validated in our reference laboratory `wasi-polyglot-bindings-reference`.”*

---

## 4. Conclusion & Transparency

AI pair-authoring is not about outsourcing thought—it is about **eliminating the friction between ideation and publication**. It allows me to maintain a dense, bilingual engineering journal while spending 95% of my mental energy where it belongs: on low-level systems engineering.

For the overarching principles governing our codebases and software runtimes, read the [Exoskeleton Manifesto](/posts/2026-08-04-exoskeleton-manifesto).

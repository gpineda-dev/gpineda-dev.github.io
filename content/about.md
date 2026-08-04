---
title: "About"
date: 2026-08-04T02:38:00+02:00
draft: false
---

# Guillaume Pineda
**SRE & Platform Engineer • Founder of Ouvrage Systems**  
📍 *Konstanz, Germany*

> Designer of minimalist and high-performance execution infrastructures. I transform technical debt and industrial legacy into declarative, resilient systems built to last.

`Brownfield / Legacy` • `Anything Declarative` • `AI Exoskeleton` • `Wasm & Systems`

---

## 🛠️ Philosophy & Commitments

### 1. Respect for the Existing (No Clean Slate)
I reject the modern reflex of wanting to rewrite everything as soon as a system is old. Working on legacy industrial codebases (active since 1996) has taught me respect for the work of the elders. I apply patterns like the *Strangler Fig*, building modern build scaffolding (CMake, Conan) around historic code to support its longevity without disrupting production.

### 2. Return to First Principles
From network protocols to Pratt parsers, I prioritize mastering theoretical fundamentals (formal grammars, graph theory) so I am never blocked by the black box of modern tools. If a proprietary tool works, I use it (Windows 11, WSL2, remote SSH). I never automate a task without first executing, auditing, and optimizing it by hand.

### 3. AI as an Exoskeleton
Artificial intelligence must be an augmenting tool for the craftsman. It helps me review code, generate boilerplate, and accelerate implementation, but control of the mental AST remains human. AI must never become an opaque oracle producing unmanaged black boxes.

---

## 🏛️ The Ouvrage Systems Ecosystem

In April 2026, at age 29, I founded **Ouvrage Systems** to group low-level, declarative tools (*Config as Data*) under a single banner for platform engineers and SREs:

* **`ouvrage-calque-go`**: Declarative templating compiler using semantic overlays (*calques*), without the complexity of mutable variables found in Helm or Jinja.
* **`ouvrage-stream-go`**: Implementation of the *Ostream* protocol, treating an infrastructure codebase as an immutable stream data bus.
* **`py-hid-declarative`**: Type-safe suite of codecs and compilers for USB HID protocols (born from the practical need to map my Thrustmaster T.16000M joystick for console gaming).

### ⚓ AML Connect: The Industrial Sandbox
To test and demonstrate these tools without violating professional NDA constraints (Siemens cRSP), I use a fictional yet realistic case study: **AML Connect (Atlantique Manutention et Levages)**. I document on this blog the step-by-step modernization of this imaginary port handling company facing 2000s-era technical debt.

---

## 📖 Background & Origins

As the eldest of five children raised in the countryside of Loir-et-Cher[^delpech], I grew up with a constant curiosity about how systems work, devouring illustrated encyclopedias (*La Grande Imagerie*) and pushing family computers to their limits.

After learning programming logic with FreePascal in middle school, self-teaching on *Site du Zéro* and *developpez.net*[^sdz], and completing CPGE MPSI/MP, I joined INSA (class of 2020). There, I developed a taste for deep upfront design (spending time at the whiteboard modeling architectures before coding) and structuring raw domain data (custom log parsers for Elastic, SSOT tracking tool using Vue.js + Google Sheets in a Junior Enterprise).

Since arriving in Konstanz in October 2020, I apply this rigor daily to critical industrial platforms.

---

[^delpech]: A nod to French singer Michel Delpech's famous song "Le Loir-et-Cher". In SRE as in Loir-et-Cher, one should never be afraid of getting their boots muddy.
[^sdz]: A nostalgic nod to Site du Zéro (SdZ) and developpez.net, the iconic French self-teaching platforms where a generation learned raw procedural PHP long before the era of heavy frameworks.

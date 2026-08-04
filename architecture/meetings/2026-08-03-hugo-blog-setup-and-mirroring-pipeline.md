# Meeting Notes: 2026-08-03 — Hugo Blog Setup, Premium PaperMod Customization & GitHub Pages Mirroring

## Metadata
* **Date**: August 3, 2026
* **Participants**: `@gpineda` (Lead Architect & Systems Engineer), `@Antigravity` (AI Coding Assistant)
* **Status**: Decided & Documented
* **Location / Repository**: `gitlab.com/gpineda-dev/g.pineda.me`

---

## 1. Context & Objectives

To document R&D discoveries, system architectures, and mathematical libraries of the Ouvrage Systems galaxy, we officially initialized the personal blog platform: **g.pineda.me** (`gitlab.com/gpineda-dev/g.pineda.me`).

Our objective was to design a blog platform that is **frugal, ultra-fast, zero-dependency**, and highly adapted to long-form systems engineering articles containing math equations and charts.

---

## 2. Technology Stack Selection (The Frugal Path)

We evaluated Static Site Generators (SSGs) under our platform principles:

*   **Adopted: GoHugo (Hugo)**
    *   *Why*: Hugo compiles to a single static Go binary. It requires **zero Node.js dependencies (no node_modules)** to build, preventing dependency rot. It offers sub-millisecond compilation times, matching our systems optimization focus ($A^*$).
*   **Rejected: Astro, VitePress, Docusaurus**
    *   *Why*: They require a complex JavaScript compilation footprint (Node.js runtime, Tailwind compilers, dozens of NPM packages) that clashes with our minimalist and frugal platform vision.

---

## 3. Theme & Customization (Premium PaperMod)

To focus on text and code block readability, we adopted the **PaperMod** theme in a git submodule, but customized it locally without modifying the submodule directory:

### 3.1 Color & Typography Customization (`assets/css/extended/custom.css`)
We replaced the default greyscale colors with a **Premium Slate (Ardoise)** color scheme:
*   *Light Mode*: Slate-50 background.
*   *Dark Mode*: Slate-900 background (`#0f172a`), Slate-800 cards (`#1e293b`).
*   *Typography*: System fonts for headers (`-apple-system`, `Segoe UI`) and optimized monospace coding fonts for code blocks.

### 3.2 Dynamic LaTeX (KaTeX) & Charts (Mermaid) Loading
To optimize loading times, we surcharge the theme layouts:
*   **KaTeX CSS/JS**: Loaded in `layouts/partials/extend_head.html` only if `math: true` is declared in the frontmatter.
*   **Mermaid JS**: Loaded in `layouts/partials/extend_footer.html` only if `mermaid: true` is declared in the frontmatter.

---

## 4. Deployment & Mirroring Pipeline

We chose a hybrid setup that keeps GitLab as the single source of truth for the codebase, while utilizing GitHub Pages as a CDN:

1.  **GitLab Source**: Code modifications are pushed to `gitlab.com/gpineda-dev/g.pineda.me`.
2.  **Git Mirroring**: GitLab repository mirroring automatically synchronizes commits to the GitHub repository `github.com/gpineda-dev/gpineda-dev.github.io`.
3.  **GitHub Actions**: Triggers on push, compiles the Hugo static files, and deploys them to GitHub Pages (`g.pineda.me`).

---
*Meeting recorded by `@Antigravity` in partnership with `@gpineda` (CPGE MPSI/MP, INSA CVL Promo 2020).*

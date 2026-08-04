---
title: "Act I: Building a Zero-BS Engineering Journal with Hugo and GitHub Pages"
date: 2026-08-03T19:00:00+02:00
draft: false
summary: "Why and how I built a minimalist, lightning-fast static journal using GoHugo, PaperMod, and GitHub Actions, setting up the baseline before moving to self-hosted Ouvrage tools."
tags: ["hugo", "devops", "platform-engineering", "gitops"]
showToc: true
---

## 1. The Case for Radical Minimalist Publishing

In a web landscape dominated by heavy client-side JavaScript frameworks, bloated CMS instances, and invasive tracking scripts, publishing technical content often comes with unacceptable overhead. 

As a Platform and Systems Engineer working with industrial legacy and low-level runtimes, my requirements for a personal journal were non-negotiable:
* **Single binary compilation**: Avoiding a multi-hundred-megabyte `node_modules` directory or ephemeral build chains just to render Markdown.
* **Zero attack surface & Zero maintenance**: No database, no PHP engine, and no Node.js runtime to emergency-patch weekly after CVE releases. Pure static Markdown is inherently secure.
* **Pure static resources (Zero infra lock-in)**: No heavy React or Vue.js client components. Clean, readable, responsive HTML/CSS capable of being served by any basic HTTP server (Apache, Nginx, S3 buckets, FTP, or directly via `file://` protocol) without needing application runtimes or reverse proxies.
* **Native bilingual support**: Seamlessly handling dual-language posts (`en` / `fr`) without fragile third-party plugins.
* **Content as Code**: Articles are fully versioned in Git and deployed deterministically via CI/CD pipelines.

This post documents **Act I**: establishing the baseline stack using **GoHugo**, the **PaperMod** theme, and **GitHub Pages**. It serves as the foundation before testing future self-hosted Ouvrage infrastructure (such as `ouvrage-lutrin`).

---

## 2. Choosing the Core Engine: GoHugo + PaperMod

### Why GoHugo?
GoHugo compiles thousands of pages in milliseconds. Written in Go, it is single-binary, memory-efficient, and imposes no complex runtime environments (no Node.js `node_modules` hell or Ruby gem dependencies).

### Theme Customization: PaperMod
PaperMod provides a clean, text-centric aesthetic with built-in dark mode and syntax highlighting via Chroma.

```yaml
# hugo.yaml snippet
markup:
  highlight:
    codeFences: true
    guessSyntax: true
    lineNos: true
    style: "monokai"
```

### 2.1 Clean Theme Overriding: Handling Mermaid and MathJax without Patching the Git Submodule
Hugo's official documentation suggests embedding the Mermaid script directly into the `baseof.html` base template. However, when using PaperMod as a Git submodule, modifying theme files directly is an anti-pattern.

Blindly applying the official recipe without accounting for PaperMod's extension points can result in a completely empty HTML `body` rendered without raising any build errors.

The elegant solution leverages Hugo's native **layout overriding hooks**:

1. **The Codeblock Render Hook** (`layouts/_markup/render-codeblock-mermaid.html`):  
   Intercepts ````mermaid ```` code blocks, escapes HTML content, and sets a page-level flag:
   ```html
   <pre class="mermaid">
     {{ .Inner | htmlEscape | safeHTML }}
   </pre>
   {{ .Page.Store.Set "hasMermaid" true }}
   ```

2. **The PaperMod Footer Extension** (`layouts/_partials/extend_footer.html`):  
   Instead of patching the theme, we inject the Mermaid ESM script only if the page contains at least one diagram:
   ```html
   {{ if .Store.Get "hasMermaid" }}
     <script type="module">
       import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.esm.min.mjs';
       mermaid.initialize({ startOnLoad: true });
     </script>
   {{ end }}
   ```

3. **The Header Hook for Math Rendering** (`layouts/_partials/extend_head.html`):  
   Similarly, conditional loading of MathJax/KaTeX is delegated to the header extension hook controlled by the `math: true` frontmatter parameter:
   ```html
   {{ if .Param "math" }}  
       {{ partialCached "math.html" . }}
   {{ end }}
   ```

This approach guarantees zero unused JavaScript overhead on non-diagram pages while leaving the theme submodule completely untouched.

### 2.2 What About Dynamic Features? (Serverless JS & Client-Side Wasm)
Static architecture does not mean an interactive dead-end. If dynamic requirements arise (full-text search, infrastructure calculators, protocol simulators), static sites handle them client-side without spinning up backend servers:

* **Indexed Full-Text Search (Fuse.js)**: PaperMod outputs a lightweight static JSON index at build time (`index.json`). Search runs 100% client-side in pure JS without requiring SQL databases or Elastic clusters.
* **Embedded Computation Engines (WASM)**: For complex algorithmic tasks (AST parsing, packet decoding), we can compile our Go modules (`ouvrage-kern-go`) into WebAssembly components and run them natively inside the browser without backend API servers.

### 2.3 Journal vs. Documentation: Why GoHugo Here, but Zensical (MkDocs) for Projects?
It is essential to separate **journal architecture** from **documentation architecture**.

* **GoHugo (For this personal journal `g.pineda.me`)**: Ideal for a chronological feed, bilingual blog posts, engineering reflections, and architectural narratives. Hugo excels at handling archives, taxonomy tags, and time-series content.
* **Zensical / Material for MkDocs (For Ouvrage Systems projects)**: Technical project documentation (like `ouvrage-doc-etl` or `ocalque`) requires strict hierarchical navigation (deep sidebar trees, API-weighted search, multi-language code tabs, native ADR integration). For this specific domain, Zensical (the modern redesign of Material for MkDocs) is our designated tool.

Applying the right tool to the right information domain is a core engineering principle: chronological for journals, hierarchical for product documentation.

---

## 3. Bilingual Directory Structure

Hugo natively supports multi-language sites by appending language tags to Markdown files. The directory structure remains flat and readable:

```text
content/
├── about.md          # English About page
├── about.fr.md       # French About page
└── posts/
    ├── 2026-08-04-building-a-zero-bs-engineering-journal-hugo.md
    └── 2026-08-04-building-a-zero-bs-engineering-journal-hugo.fr.md
```

Configuration in `hugo.yaml`:
```yaml
defaultContentLanguage: "en"

languages:
  en:
    languageCode: "en-us"
    weight: 1
    title: "g.pineda.me"
  fr:
    languageCode: "fr-fr"
    weight: 2
    title: "g.pineda.me"
```

---

## 4. GitOps Deployment Pipelines (GitHub Pages & Legacy FTP)

The key advantage of a static site generator is total delivery freedom. The deployment pipeline can adapt to any target environment.

### Option A: GitHub Actions Pipeline for GitHub Pages (The Selected Approach)

On every `git push` to the `main` branch, a native GitHub Actions workflow compiles the static site and deploys it directly to GitHub Pages in two isolated jobs (*build* and *deploy*):

```yaml
name: Deploy Hugo Site to GitHub Pages

on:
  push:
    branches: ["main"]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build Static Files
        run: hugo --minify

      - name: Upload Artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Option B: GitLab CI Pipeline + FTP Sync (`lftp`) (Classic Alternative)

For traditional web hosts or VPS environments lacking native GitOps support, a GitLab CI job leveraging `lftp` with an incremental mirror (`--delete`) provides clean, automated delivery:

```yaml
stages:
  - build
  - deploy

# Step 1: Static Site Compilation
hugo:
  stage: build
  image: jguyomard/hugo-builder
  script:
    - hugo
  artifacts:
    paths:
      - public

# Step 2: Incremental Delta Sync via FTP
ftp:
  stage: deploy
  image: ubuntu:18.04
  before_script:
    - apt-get update -qy
    - apt-get install -y lftp
  script:
    - lftp -e "open $FTP_SITE; user $FTP_USERNAME $FTP_PASSWORD; mirror -X .* -X .*/ --reverse --verbose --delete public/ /www/guillaume/; bye"
  dependencies:
    - hugo
```

This flexibility allows engineers to maintain a 100% local, versioned writing workflow while guaranteeing deterministic deployments to modern CDNs or classic FTP targets alike.

---

## 5. What's Next: The Path to `ouvrage-lutrin`

This baseline setup fulfills all immediate needs for speed, reliability, and security. However, as part of the **Ouvrage Systems** initiative, this site will eventually transition to **Act II**:
* Hosting static assets directly from S3/MinIO buckets.
* Serving traffic via **`ouvrage-lutrin`**, a lightweight edge gateway reading routing rules dynamically from YAML manifests.

Until then, GitHub Pages provides a rock-solid, zero-maintenance foundation for publishing deep technical content.

---

> 🛠️ *This post was co-designed with Gemini (Antigravity) using our [Augmented Pair-Authoring Workflow](/posts/2026-08-04-augmented-pair-authoring-workflow) (see also our [Exoskeleton Manifesto](/posts/2026-08-04-exoskeleton-manifesto)).*



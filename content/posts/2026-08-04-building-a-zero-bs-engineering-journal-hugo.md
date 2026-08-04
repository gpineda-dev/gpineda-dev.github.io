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

-> https://gohugo.io/host-and-deploy/host-on-github-pages/
```yaml
name: Build and deploy
on:
  push:
    branches:
      - main
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
defaults:
  run:
    shell: bash
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      # Define tool versions
      DART_SASS_VERSION: 1.101.0
      GO_VERSION: 1.26.4
      HUGO_VERSION: 0.164.0
      NODE_VERSION: 24.18.0

      # Set the build time zone
      TZ: Europe/Oslo
    steps:
      - name: Checkout
        uses: actions/checkout@v7
        with:
          submodules: recursive
          fetch-depth: 0
          lfs: false

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v6

      - name: Create a local tools directory
        run: |
          mkdir -p "${HOME}/.local"

      - name: Install Go
        if: hashFiles('go.mod') != ''
        uses: actions/setup-go@v6
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: false

      - name: Install Node.js
        if: hashFiles('package-lock.json') != ''
        uses: actions/setup-node@v6
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Install Dart Sass
        run: |
          echo "Installing Dart Sass ${DART_SASS_VERSION}..."
          curl -sfL --output-dir "${{ runner.temp }}" -O "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          tar -C "${HOME}/.local" -xf "${{ runner.temp }}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          echo "${HOME}/.local/dart-sass" >> "${GITHUB_PATH}"

      - name: Install Hugo
        run: |
          echo "Installing Hugo ${HUGO_VERSION}..."
          curl -sfL --output-dir "${{ runner.temp }}" -O "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir "${HOME}/.local/hugo"
          tar -C "${HOME}/.local/hugo" -xf "${{ runner.temp }}/hugo_${HUGO_VERSION}_linux-amd64.tar.gz"
          echo "${HOME}/.local/hugo" >> "${GITHUB_PATH}"

      - name: Log tool versions
        run: |
          echo "Logging tool versions..."
          command -v sass &> /dev/null && echo "Dart Sass: $(sass --version)" || echo "Dart Sass: not installed"
          command -v go &> /dev/null && echo "Go: $(go version)" || echo "Go: not installed"
          command -v hugo &> /dev/null && echo "Hugo: $(hugo version)" || echo "Hugo: not installed"
          command -v node &> /dev/null && echo "Node.js: $(node --version)" || echo "Node.js: not installed"

      - name: Configure Git
        run: |
          echo "Configuring Git..."
          git config --global core.quotepath false

      - name: Fetch full Git history
        run: |
          if [[ $(git rev-parse --is-shallow-repository) == true ]]; then
            echo "Fetching full Git history..."
            git fetch --unshallow
          fi

      - name: Initialize Git submodules
        run: |
          if [[ -f .gitmodules ]]; then
            echo "Initializing Git submodules..."
            git submodule update --init --recursive
          fi

      - name: Install Node.js dependencies
        run: |
          if [[ -f package-lock.json ]]; then
            echo "Installing Node.js dependencies..."
            npm ci
          fi

      - name: Cache restore
        id: cache-restore
        uses: actions/cache/restore@v6
        with:
          path: ${{ runner.temp }}/.cache/hugo
          key: hugo-${{ github.run_id }}
          restore-keys: hugo-

      - name: Build
        run: |
          echo "Building the project..."
          hugo build \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/" \
            --cacheDir "${{ runner.temp }}/.cache/hugo"

      - name: Cache save
        uses: actions/cache/save@v6
        with:
          path: ${{ runner.temp }}/.cache/hugo
          key: ${{ steps.cache-restore.outputs.cache-primary-key }}

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v5
        with:
          include-hidden-files: false
          path: ./public
  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
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



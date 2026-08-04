---
title: "Acte I : Construire un journal d'ingénierie sans blabla avec Hugo et GitHub Pages"
date: 2026-08-03T19:00:00+02:00
draft: false
summary: "Pourquoi et comment j'ai mis en place ce site statique minimaliste et ultra-rapide avec GoHugo, PaperMod et GitHub Actions, posant les jalons avant de basculer sur nos propres outils d'infra Ouvrage."
tags: ["hugo", "devops", "platform-engineering", "gitops"]
showToc: true
---

## 1. Pourquoi refuser l'over-engineering ?

Les blogs sous Next.js ou WordPress pèsent fréquemment plusieurs mégaoctets pour afficher quelques paragraphes de texte. En tant qu'ingénieur plateforme travaillant sur des systèmes industriels bas-niveau et du legacy, mon cahier des charges pour ce journal était mesuré :

* **Un binaire unique et rapide** : Éviter un dossier `node_modules` de plusieurs centaines de mégaoctets ou des dépendances éphémères juste pour compiler du Markdown.
* **Surface d'attaque zéro & Zéro maintenance** : Pas de base de données, pas de moteur PHP ou Node.js à patcher chaque semaine en urgence suite à une CVE. Le Markdown statique est intrinsèquement invulnérable.
* **La ressource pure (Zéro dépendance d'infra)** : Pas de composants React ou Vue.js lourds. Du HTML/CSS brut, lisible et responsive, capable d'être servi par n'importe quel serveur HTTP (Apache, Nginx, bucket S3, FTP, ou même directement via le protocole `file://`) sans nécessiter de reverse-proxy ou d'application serveur complexe.
* **Multilingue natif** : Basculer entre français (`.fr.md`) et anglais (`.md`) sans ajouter une suite de plugins fragiles.
* **Tout sous Git** : Le contenu est traité comme du code, versionné et déployé de manière déterministe par CI/CD.

Cet article pose l'**Acte I** : mon setup de départ avec **GoHugo**, le thème **PaperMod** et **GitHub Pages**. C'est ma ligne de base avant d'aller tester mes propres briques d'infra auto-hébergées Ouvrage (comme `ouvrage-lutrin`).

---

## 2. Le moteur : GoHugo + PaperMod

### Pourquoi GoHugo ?
GoHugo compile des milliers de pages en quelques millisecondes. Écrit en Go, c'est un binaire unique et léger. Il consomme très peu de mémoire et conserve des builds stables à travers le temps.

### Personnalisation : Thème PaperMod
J'ai choisi PaperMod pour son esthétique sobre centrée sur le texte, son mode sombre et la coloration syntaxique Chroma intégrée.

```yaml
# Extrait de hugo.yaml
markup:
  highlight:
    codeFences: true
    guessSyntax: true
    lineNos: true
    style: "monokai"
```

### 2.1 Le piège du rendu Mermaid & MathJax (Overriding propre sans altérer le sous-module)
C'est ici qu'il faut être vigilant. La doc officielle d'Hugo préconise d'inclure le script Mermaid dans le modèle `baseof.html` avec un petit `.Store.Get "hasMermaid"`. 

Sauf qu'en pratique avec PaperMod (qu'on garde sous forme de submodule Git propre pour éviter la dette), si tu suis bêtement la doc officielle... **tu te retrouves avec une page au HTML désespérément vide (`<body></body>`), sans le moindre message d'erreur au build.** C'est le genre de bug silencieux qui te fait perdre 20 minutes à chercher pourquoi ton serveur dev ne recrache rien.

La solution propre consiste à utiliser les hooks de surcharge natifs d'Hugo et les partials d'extension de PaperMod :

1. **Le Codeblock Render Hook** (`layouts/_markup/render-codeblock-mermaid.html`) :  
   Il intercepte les blocs ````mermaid ````, échappe le HTML et lève le drapeau global :
   ```html
   <pre class="mermaid">
     {{ .Inner | htmlEscape | safeHTML }}
   </pre>
   {{ .Page.Store.Set "hasMermaid" true }}
   ```

2. **La surcharge de footer PaperMod** (`layouts/_partials/extend_footer.html`) :  
   Plutôt que d'aller hacker le thème, on surcharge le partial de footer pour n'injecter le script ESM Mermaid **que si la page contient réellement un diagramme** :
   ```html
   {{ if .Store.Get "hasMermaid" }}
     <script type="module">
       import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.esm.min.mjs';
       mermaid.initialize({ startOnLoad: true });
     </script>
   {{ end }}
   ```

3. **Le hook d'en-tête pour les maths** (`layouts/_partials/extend_head.html`) :  
   Même punition pour MathJax : injection uniquement si la page déclare `math: true` dans son en-tête :
   ```html
   {{ if .Param "math" }}  
       {{ partialCached "math.html" . }}
   {{ end }}
   ```

Résultat : zéro JS inutile sur les pages de texte pur, et un submodule PaperMod qui reste 100% intact.

### 2.2 Et si on a besoin de dynamique ? (Serverless JS & Wasm client-side)
Statut statique ne veut pas dire site figé. En cas de besoin de fonctionnalités interactives (recherche plein texte, calculettes d'infrastructure, simulateurs de protocoles), l'architecture statique permet d'injecter du dynamique sans installer de backend :

* **Recherche Plein Texte Indexée (Fuse.js)** : PaperMod génère un simple index JSON à la compilation (`index.json`). La recherche s'exécute à 100% côté client, en pur JS, sans base de données SQL ou cluster Elastic.
* **Moteurs de calcul embarqués (WASM)** : Si l'on souhaite faire tourner des algorithmes complexes (parseurs d'AST, décodeurs de paquets), on peut directement compiler nos modules Go (`ouvrage-kern-go`) en composants WebAssembly et les exécuter dans le navigateur sans aucun serveur backend.

### 2.3 Blog vs Documentation : Pourquoi GoHugo ici, mais Zensical (MkDocs) pour les projets ?
Il est important de séparer l'**architecture de journal** de l’**architecture de documentation**. 

* **GoHugo (Pour ce journal personal `g.pineda.me`)** : Idéal pour un fil chronologique, un blog bilingue, des réflexions d'ingénierie et des récits d'architecture. Hugo excelle dans la gestion des archives, des tags et des flux temporels.
* **Zensical / Material for MkDocs (Pour la documentation d'Ouvrage Systems)** : La documentation technique de nos projets (comme `ouvrage-doc-etl` ou `ocalque`) requiert une navigation hiérarchique stricte (arbre de navigation à plusieurs niveaux, recherche pondérée sur la doc API, onglets de code multi-langages, intégration des ADRs). Pour cet usage, Zensical (la refonte moderne de Material for MkDocs) est l'outil spécialisé retenu.

Utiliser le bon outil pour le bon type d'information est un principe clé d'ingénierie : du chronologique pour le journal, du hiérarchique pour la documentation produit.

---

## 3. Structure de dossiers bilingue

Hugo gère nativement le multilingue en ajoutant le suffixe de langue aux fichiers Markdown. La structure reste simple et à plat :

```text
content/
├── about.md          # Page À propos en anglais
├── about.fr.md       # Page À propos en français
└── posts/
    ├── 2026-08-04-building-a-zero-bs-engineering-journal-hugo.md
    └── 2026-08-04-building-a-zero-bs-engineering-journal-hugo.fr.md
```

Configuration dans `hugo.yaml` :
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

## 4. Pipelines de déploiement GitOps (GitHub Pages & FTP Legacy)

L'avantage d'un générateur de site statique est la liberté totale du mode de livraison. Le pipeline de déploiement peut s'adapter à n'importe quel environnement d'hébergement.

### Option A : Pipeline GitHub Actions pour GitHub Pages (L'approche retenue)

À chaque `git push` sur la branche `main`, un workflow natif GitHub Actions compile le site statique et le déploie directement sur GitHub Pages en deux étapes isolées (*build* puis *deploy*) :

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

### Option B : Pipeline GitLab CI + Sync FTP (`lftp`) (L'approche old-school mais increvable)

Pour les serveurs hébergés chez des prestataires traditionnels (mutualisés, VPS) sans support GitOps moderne, on peut utiliser un job GitLab CI articulé autour de `lftp` avec un miroir incrémental (`--delete`) :

```yaml
stages:
  - build
  - deploy

# Étape 1 : Compilation du site statique
hugo:
  stage: build
  image: jguyomard/hugo-builder
  script:
    - hugo
  artifacts:
    paths:
      - public

# Étape 2 : Synchronisation delta via FTP
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

L'avantage de cette approche ? On garde un workflow de rédaction 100% versionné en local, tout en poussant les modifs de manière déterministe, que ce soit vers un CDN cloud-native ou vers un serveur FTP traditionnel.

---

## 5. Et ensuite ? La route vers `ouvrage-lutrin`

Ce setup V1 remplit parfaitement son contrat : c'est rapide, c'est gratuit, c'est sécurisé et ça ne demande zéro maintenance. 

Mais comme tout SRE qui aime construire ses propres outils, ce site servira de cobaye pour l'**Acte II** d'**Ouvrage Systems** :
* Stocker les assets statiques directement sur des buckets S3/MinIO.
* Servir le trafic via **`ouvrage-lutrin`**, un routeur edge léger lisant ses règles de routage dynamiquement depuis des manifestes YAML.

D'ici là, GitHub Pages fait le job sans rechigner.

---

> 🛠️ *Cet article a été co-conçu avec Gemini (Antigravity) selon notre [Workflow de Rédaction Augmentée](/fr/posts/2026-08-04-augmented-pair-authoring-workflow) (voir aussi notre [Manifeste de l'Exosquelette](/fr/posts/2026-08-04-exoskeleton-manifesto)).*




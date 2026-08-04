---
title: "De la note brute à l'article : Mon workflow de rédaction augmentée en pair-programming"
date: 2026-08-04T03:15:00+02:00
draft: false
summary: "Une analyse pratique de ma méthode de collaboration avec les LLMs pour traduire mes notes d'ingénierie brutes en articles bilingues structurés sans perdre l'authenticité de ma voix."
tags: ["ai", "meta", "writing", "workflow"]
showToc: true
---

## 1. Pourquoi séparer la génération de code de la rédaction ?

Dans notre [Manifeste de l'Exosquelette](/fr/posts/2026-08-04-exoskeleton-manifesto), nous avons établi la séparation stricte entre l'architecture humaine et l'implémentation machine. Cependant, appliquer l'IA à la **rédaction technique** présente un défi différent de la génération de code :

* **En Génération de Code** : Le résultat est validé par un compilateur, des tests unitaires et des métriques d'exécution. La justesse est binaire.
* **En Rédaction de Prose** : Le résultat doit préserver l'authenticité humaine, l'historique personnel et les nuances d'ingénierie. Une prose IA naïve produit du texte générique, creux et sur-marketé.

Pour éviter cela, j'utilise un **Workflow de Rédaction Augmentée** où l'IA agit strictement comme éditeur et traducteur, tandis que je reste la seule source de vérité technique et narrative.

---

## 2. La division du travail

```text
[ Guillaume (@gpineda) ]                                 [ IA Exosquelette ]
───────────────────────                                 ──────────────────
Idées brutes & anecdotes  ─────( 1. Input brut )─────►   Structure & Catégorisation
Contraintes techniques     ◄───( 2. Clarification )───   Mise en forme Markdown
Veto final & Verification ─────( 3. Validation )─────►   Traduction bilingue
```

1. **La Source Humaine** : Fournit les notes brutes, les choix techniques, les anecdotes historiques et les contraintes métier.
2. **L'Exosquelette** : Organise les puces, propose une mise en page progressive (*progressive disclosure*), souligne les trous logiques et gère la traduction bilingue Markdown (`en` / `fr`).
3. **Le Veto Humain** : Recompose chaque ligne, purge l'exagération de l'IA et garantit une adéquation à 100% avec les faits réels.

---

## 3. Étude de cas pas-à-pas : Du besoin système à l'architecture WASM

Pour illustrer ce workflow, voici la trace d'une session réelle où un besoin d'ingénierie complexe (conception du moteur `ocalque` et compilation WASM de `ouvrage-kern-go`) est traduit en un article d'architecture fluide.

### Étape 1 : L'Input Humain brut (`@gpineda`)
Un flux de pensées techniques denses décrivant un problème d'outillage et de compilateur :
> *"Pour ma codebase legacy AML Connect sous Visual Studio Express, je ne veux pas d'un moteur de remplacement de texte type Jinja qui corrompt le code. Il me faut `ocalque`, un moteur de templates AOT utilisant le Shadow Pattern (directives cachées dans des commentaires) pour que le code reste propre et compilable en dev. Le cœur du parseur de Pratt multi-grammaires (`pkg/lang` dans `ouvrage-kern-go`) est en pur Go sans dépendance. Le CLI `oklang` le pilote, mais le package `pkg/lang` doit être compilé en WASI Preview 2 pour embarquer ce moteur de Pratt (construction d'AST, évaluation, export Mermaid/DOT) nativement en Python et JS sans tout réécrire. D'où le lab `wasi-polyglot-bindings-reference` ouvert cette semaine."*

### Étape 2 : Le cadrage & structuration par l'Exosquelette IA
L'IA identifie l'arbre des responsabilités et propose de découper le sujet pour le lecteur :
> *"L'intention est d'une grande clarté architecturale. Ne mélangeons pas le CLI (`oklang`) et la bibliothèque cœur (`pkg/lang`). Isolons trois niveaux : 1) Le besoin produit (Shadow Pattern pour dev C sans pollution), 2) La solution théorique (Moteur de Pratt multi-grammaires découpé pour générer des AST/Mermaid/DOT), et 3) L'export polyglotte (compilation de `pkg/lang` en WASM/WASI Preview 2 pour cibler Python/Node.js). Structureurs cela sous forme de schéma de flux."*

### Étape 3 : Le résultat de prose technique validé
Le paragraphe d'architecture final, rédigé et relu par l'ingénieur :
> *« Afin d'évaluer les directives du moteur de templates AOT `ocalque` sans polluer le code C d'origine sous les IDEs historiques (le Shadow Pattern), nous avons isolé son moteur de parsing dans `ouvrage-kern-go` (`pkg/lang`). Conçu en pur Go sans dépendances, ce package implémente un moteur de Pratt multi-grammaires capable de construire des AST, d'évaluer des expressions et d'exporter des représentations de graphes (Mermaid, DOT, GraphML). Pour réutiliser ce cœur algorithmique dans d'autres écosystèmes (outillage Python, serveurs LSP en TypeScript) sans réécrire le parseur, `pkg/lang` est compilé en composant WebAssembly (WASI Preview 2), un composant testé dans notre laboratoire de référence `wasi-polyglot-bindings-reference`. »*

---

## 4. Conclusion & Transparence

La rédaction augmentée avec une IA ne consiste pas à sous-traiter sa pensée — elle vise à **éliminer la friction entre l'idée et la publication**. Elle me permet de maintenir un journal d'ingénierie bilingue et dense tout en consacrant 95% de mon énergie mentale là où elle doit être : dans l'ingénierie système bas-niveau.

Pour comprendre les principes généraux qui régissent nos codebases et nos runtimes logicielle, lisez le [Manifeste de l'Exosquelette](/fr/posts/2026-08-04-exoskeleton-manifesto).

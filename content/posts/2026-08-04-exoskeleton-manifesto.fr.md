---
title: "Le Manifeste de l'Exosquelette : L'ingénierie augmentée sans l'illusion de l'oracle"
date: 2026-08-04T03:09:00+02:00
draft: false
summary: "Comment j'utilise les LLMs comme un exosquelette de pair-programming pour accélérer l'implémentation tout en conservant 100% de maîtrise humaine sur l'architecture et le déterminisme."
tags: ["ai", "manifesto", "software-craftsmanship", "platform-engineering"]
showToc: true
---

## 1. La dichotomie : L'Oracle vs L'Exosquelette

L'Intelligence Artificielle dans l'ingénierie logicielle se divise aujourd'hui en deux paradigmes opposés :

* **Le paradoxe de l'Oracle** : Attendre du LLM qu'il agisse comme une autorité omnisciente. Les développeurs expriment des besoins flous et copient-collent aveuglément du code généré. Cela produit des boîtes noires fragiles, de la dette technique ingérable et une atrophie de l'intuition d'ingénieur.
* **Le paradigme de l'Exosquelette** : Traiter le LLM comme une armure assistée à haut débit. L'ingénieur humain conserve la propriété totale de l'architecture système, des contraintes aux limites et de la rigueur structurelle. L'IA agit comme un relecteur impitoyable, un générateurs de boilerplate et un binôme de pair-programming.

Chez **Ouvrage Systems**, nous rejetons l'Oracle. Nous adoptons l'Exosquelette.

```text
[ Ingénieur Humain ] ===( Contraintes d'architecture & AST mental )===> [ IA Exosquelette ]
        ▲                                                                     │
        │                                                                     ▼
[ Déterminisme strict & Audit ] <===( Boilerplate & Brouillons relus )========┘
```

---

## 2. Notre workflow de co-conception

Co-concevoir du logiciel et du contenu technique avec un exosquelette IA repose sur trois principes stricts :

1. **L'AST mental humain d'abord** : Aucun code ni article n'est généré sans un modèle mental préalable explicite. Les réunions d'architecture, les schémas au tableau blanc et les invariants spatiaux sont définis par l'ingénieur avant tout prompt.
2. **Décharge du boilerplate** : L'exosquelette excelle dans le travail répétitif — traduire des AST mentaux en structures Go denses, générer de la documentation Markdown multilingue ou écrire des suites de tests unitaires.
3. **Revue impitoyable & Refus** : Toute abstraction hallucinée ou dépendance inutile introduite par l'IA est immédiatement rejetée. L'ingénieur humain reste le seul décideur des commits Git.

---

## 3. Transparence & Attribution

Chaque article et projet publié sur `g.pineda.me` et la constellation **Ouvrage Systems** respecte cette charte :

> 🛠️ **Transparence** : Les articles et codebases de ce site sont architecturés par Guillaume Pineda (`@gpineda`) et exécutés en pair-programming avec des exosquelettes IA (tels que Google Antigravity / Gemini). Les décisions architecturales, le design système et la validation finale restent 100% maîtrisés par l'humain.

En couplant l'intuition humaine avec le débit de la machine, nous bâtissons du logiciel conçu pour durer +10 ans tout en maintenant une transparence totale envers nos lecteurs.

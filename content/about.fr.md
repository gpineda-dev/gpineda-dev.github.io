---
title: "À propos"
date: 2026-08-04T02:38:00+02:00
draft: false
---

# Guillaume Pineda
**SRE & Platform Engineer • Fondateur d'Ouvrage Systems**  
📍 *Konstanz, Allemagne*

> Concepteur d'infrastructures d'exécution minimalistes et performantes. Je transforme la dette technique et le legacy industriel en systèmes déclaratifs, résilients et conçus pour durer.

`Brownfield / Legacy` • `Anything Declarative` • `IA Exosquelette` • `Wasm & Systems`

---

## 🛠️ Philosophie & Engagements

### 1. Le Respect de l'Existant (Pas de table rase)
Je refuse le réflexe moderne de vouloir tout réécrire dès qu'un système est ancien. Travailler sur des codebases industrielles héritées (actives depuis 1996) m'a appris le respect du travail des anciens. J'applique des patrons comme le *Strangler Fig*, en construisant des outils de build modernes (CMake, Conan) autour du code historique pour accompagner sa longévité sans perturber la production.

### 2. Le Retour aux Principes Premiers
Du protocole réseau au parseur de Pratt, je privilégie la maîtrise des fondamentaux théoriques (grammaires formelles, théorie des graphes) pour ne jamais être bloqué par la boîte noire des outils modernes. Si un outil propriétaire fonctionne, je l'utilise (Windows 11, WSL2, SSH). Je n'automatise jamais une tâche sans l'avoir d'abord exécutée manuellement, auditée et optimisée à la main.

### 3. L'IA comme Exosquelette
L'intelligence artificielle doit être un outil d'augmentation pour l'artisan. Elle m'aide à relire le code, générer le boilerplate et accélérer l'implémentation, mais le contrôle de l'AST mental reste humain. L'IA ne doit jamais devenir un oracle opaque qui produit des boîtes noires incomprises.

---

## 🏛️ L'Écosystème Ouvrage Systems

En avril 2026, à 29 ans, j'ai fondé **Ouvrage Systems** pour regrouper sous une même bannière des outils bas-niveau et déclaratifs (*Config as Data*) dédiés aux ingénieurs plateforme et SRE :

* **`ouvrage-calque-go`** : Compilateur de templates déclaratifs sous forme d'overlays sémantiques (*calques*), sans la complexité des variables mutables de Helm ou Jinja.
* **`ouvrage-stream-go`** : Implémentation du protocole *Ostream*, traitant une base de code d'infrastructure comme un bus de données immuable.
* **`py-hid-declarative`** : Suite de codecs et compilateurs type-safe pour les protocoles USB HID (né du besoin de mapper mon joystick Thrustmaster T.16000M pour la console).

### ⚓ AML Connect : Le Bac à Sable Industriel
Pour tester et illustrer ces outils sans enfreindre mes clauses de confidentialité professionnelles (Siemens cRSP), j'utilise un cas d'étude fictif mais réaliste : **AML Connect (Atlantique Manutention et Levages)**. Je documente sur ce blog la modernisation pas-à-pas de cette entreprise portuaire imaginaire confrontée à la dette technique des années 2000.

---

## 📖 Parcours & Origines

Aîné d'une fratrie de cinq enfants élevée dans les campagnes du Loir-et-Cher[^delpech], j'ai grandi avec une curiosité constante pour le fonctionnement des systèmes, dévorant les encyclopédies illustrées (*La Grande Imagerie*) et poussant l'informatique familiale dans ses retranchements. 

Après l'apprentissage de la logique avec FreePascal au collège, l'auto-formation sur le *Site du Zéro* et *developpez.net*[^sdz], et une CPGE MPSI/MP, j'ai rejoint l'INSA (promo 2020). C'est là que j'ai développé mon goût pour la conception poussée (passer du temps devant le tableau blanc à modéliser les architectures avant de coder) et pour la structuration de données métiers brutes (custom log parsers pour Elastic, outil de suivi SSOT Vue.js + Google Sheets en Junior Entreprise).

Depuis mon arrivée à Konstanz en octobre 2020, j'applique cette rigueur au quotidien sur des plateformes industrielles critiques.

---

[^delpech]: Clin d'œil à la célèbre chanson de Michel Delpech. Car en ingénierie système *brownfield* comme dans le Loir-et-Cher, il ne faut jamais qu'il y ait de gêne à *marcher dans la boue*.
[^sdz]: Une pensée nostalgique pour l'époque du Site du Zéro (SdZ) et de developpez.net, où l'on apprenait le PHP procédural brut à coup de `mysql_query()` bien avant l'avènement des frameworks opulents.

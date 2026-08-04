# Meeting: 2026-08-04 - Archéologie Système : La Genèse d'Ouvrage Systems (2017-2026)

## Métadonnées
* **Date** : 4 août 2026
* **Participants** : `@gpineda` (Guillaume Pineda - Fondateur d'Ouvrage Systems), `@Gemini` (AI Exoskeleton)
* **Objet** : Exhumation des archives logicielles et traçabilité chronologique des intuitions d'ingénierie (Data-Centric ERP, Document-as-Code, AST Parsers, Contextus) depuis les années INSA jusqu'à la création d'Ouvrage Systems.

---

## 1. Chronologie Réelle & Évolution des Projets

```mermaid
timeline
    title L'Évolution d'Ouvrage Systems (2017-2026)
    section 2017-2020 : INSA & Junior Entreprise
        2017 : Entrée Pôle Qualité IRIS (Constat cécité ERP Arpège)
        2019 Mai : simpleJsFileParser (Reverse EML Zimbra & Wekan SLA)
        2019 Juin : Patch Google Script + Vue.js (SSOT Échéances)
        2019 Oct : LinkedCompass (Business Plan ERP Modulaire & Storage Agnostique)
        2020 Fév : Fin mandat Responsable Qualité
    section 2020-2021 : Post-Diplôme & Stage
        2020 Mars : Stage DevOps Konstanz (Siemens cRSP)
        2020 Avril : QualityDashboard (SaaS Django REST / Vuetify / BPMN2 sur Heroku)
        2020 Oct : Prise de poste Ingénieur Système (Siemens cRSP Konstanz)
        2021 Avril : Brouillon Medium "Doc-as-Code" + POC yaml-documents & document-filler
    section 2025-2026 : Maturation & Ouvrage
        2025 Déc : Congés R&D Gemini Pro (Théorisation d'Ouvrage & Contextus)
        2026 Avril : Création Ouvrage Systems (ocalque, okern, WASI Polyglot)
        2026 Août : Refonte g.pineda.me (Hugo, PaperMod, Manifeste Exosquelette)
```

---

## 2. Analyse des Pièces d'Archéologie Exhumées

### 2.1 `simpleJsFileParser` (Mai 2019)
* **Contexte** : Absence d'API REST sur le webmail universitaire Zimbra/Renater de l'INSA.
* **Ingénierie** : Reverse-engineering client-side des fichiers `.eml` et des exports JSON Wekan (Kanban Meteor).
* **Innovation** : 
  * Reconstitution d'échanges de mails par `parent_id` (`EmlExchange.js`).
  * Calcul automatique des SLA et temps de traitement (`calcDelay()`, `_e.time = (endAt - receivedAt)`).
  * Packaging browserified sur CDN jsDelivr pour exécution 100% navigateur sans backend.

### 2.2 `LinkedCompass` (Octobre 2019 - Février 2020)
* **Contexte** : Projet entreprenarial de création d'entreprise de fin d'études INSA.
* **Vision** : ERP modulaire pour TPE/PME basé sur des microservices Go (Docker/Kubernetes).
* **Pépites d'Architecture** :
  * **Storage Agnostique** : Moteur de fichiers capable de dialoguer de façon transparente avec GDrive, Dropbox, FTP/SSH ou du stockage local.
  * **No Clean Slate** : Refus d'imposer une réécriture d'infra aux PME ; interfaçage avec les SI existants.
  * **Pilotage par les processus (BPMN)** : Intégration de capteurs de workflows et alertes automatiques sur dépassement de deadlines.

### 2.3 `QualityDashboard` (Avril 2020)
* **Contexte** : Tentative de SaaS de suivi qualimétrique/documentaire sous Django REST + Vuetify sur Heroku.
* **Fonctionnalités avancées** :
  * Intégration TDD & GitLab CI pour la validation des livrables informatiques.
  * Workflow de revue & d'annotations de documents (suggestions, pings, preuves de correction).
  * Édition graphique BPMN2 et routage de règles décisionnelles.
* **Constat** : Abandonné car trop lourd à maintenir seul sans équipe dev dédiée.

### 2.4 `yaml-documents` & `document-filler` (Avril 2021)
* **Contexte** : Bascule du modèle SaaS vers la philosophie **Document-as-Code** (GitLab CI).
* **Découverte majeure (`schema_parser.js`)** :
  * Spécification YAML déterministe avec contraintes (`alert: on: out_of_range`).
  * Extension de JSON Schema via **AJV** avec mot-clé personnalisé `computed` et priorisation DAG des dépendances (`priority`).
  * Résolution des chemins JSONPath relatifs (`@.`) et absolus (`$.`).
  * Évaluation isolée d'expressions dynamiques via un bac à sable JS (`compileCode` avec `Proxy` et `Symbol.unscopables`).

---

## 3. La Convergence vers Ouvrage Systems & Contextus (2025-2026)

L'analyse de ces 9 ans d'archives montre que la galaxie **Ouvrage Systems** est la maturation directe de trois obsessions constantes :

1. **Le Document Data-Centric comme SSOT** : L'événement ou l'acte juridique scellé (`PublishedDocument`) est la vérité brute ; l'état de l'ERP n'est qu'une vue matérialisée dérivée (*Event Sourcing Scellé*).
2. **Du JS Sandbox à WebAssembly (`okern` / WASI)** : Remplacer l'évaluation JS fragile de 2021 par un parseur de Pratt déterministe en pur Go (`pkg/lang`), compilé en composant **WASI Preview 2** pour une distribution polyglotte zéro-dépendance.
3. **Le Protocole `Contextus` (Le "Git de la Réalité")** : L'extension globale du concept d'assertion signée (CBOR, DID, KCL, Trust Boundaries, ZKP) pour créer un marché de la donnée certifiée à la source face au bruit de l'IA générative.

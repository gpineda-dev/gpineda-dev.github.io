# FlowWeaver (FlowWeaver-Go) - Design Document & Specification

## Métadonnées
* **Projet** : `FlowWeaver` (`flow-weaver-go`)
* **Auteur** : Guillaume Pineda (`@gpineda-dev`)
* **Statut** : Document d'architecture & Spécification formelle (Engine Workflow As-Code)
* **Licence** : MIT
* **Rôle Écosystème** : Moteur d'orchestration déclaratif embarqué pour `ProtocolSmith` et `Ouvrage Systems`.

---

## 1. Vision & Modèle Théorique

`FlowWeaver` est un **moteur de workflow déclaratif embarqué ("As-Code")** sous forme de bibliothèque Go pure. Il comble le vide entre les petites bibliothèques FSM simplistes et les grosses plateformes d'orchestration distribuées (style Temporal ou Camunda).

### Les 4 Piliers d'Architecture :
1. **Déclaratif** : Description du *Quoi* (YAML/CUE), l'exécuteur s'occupe du *Comment*.
2. **Embarqué (Embeddable)** : Une bibliothèque Go de haut niveau sans dépendances d'infrastructure lourde.
3. **Modèle "Bring Your Own Actions"** : Totalement agnostique du domaine métiers (`Action` interface Go).
4. **Moteur à deux phases (Build & Run)** :
   * **Phase 1 : Build (Offline/Compile-Time)** : Résolution des dépendances (DAG), détection de cycles (DFS), validation du schéma CUE (`cue.Value`), et compilation en un binaire d'états immuables (`bundle.yml`).
   * **Phase 2 : Run (Online/Runtime)** : Parsing du `bundle.yml` en automate d'état fini (FSM) ultra-performant et exécuté sur des `ExecutionContext` isolés par goroutines.

---

## 2. Spécification de la Grammaire (Fichiers `.flow.yml`)

Un fichier `.flow.yml` définit une bibliothèque autonome de composants réutilisables :
* `package` : Définition du namespace et des exports publics.
* `imports` : Ingestion des bibliothèques externes avec alias (`as`).
* `instructions` : Instructions nommées (appel aux `Action` Go).
* `flows` : Séquences d'étapes réutilisables.
* `workflow` : L'orchestrateur de haut niveau.

```mermaid
graph TD
    subgraph PhaseBuild ["Phase 1 : Build (Offline / Off-line Compiler)"]
        YAML[".flow.yml Source Files"]
        CUE["CUE Schema Validation (go:embed)"]
        DAG["Dependency Resolution & Cycle Detection (DFS)"]
        BUNDLE["bundle.yml (Compiled Bytecode)"]
        
        YAML --> CUE --> DAG --> BUNDLE
    end

    subgraph PhaseRun ["Phase 2 : Run (Online Execution Engine)"]
        BUNDLE --> FSM["In-Memory State Machine (FSM / looplab)"]
        HOST["Host Application (ProtocolSmith / Ouvrage)"]
        ACTIONS["Registered Go Actions (BYO Actions)"]
        
        HOST -->|weaver.NewEngine()| FSM
        ACTIONS -->|Execute(ctx)| FSM
    end
```

---

## 3. Emboîtement dans la Galaxie Ouvrage & ProtocolSmith

`FlowWeaver` fournit le **cerveau d'orchestration**, tandis que l'application hôte (`ProtocolSmith`, `ouvrage-doc-etl`, etc.) fournit les **mains** (les actions Go métier) :

* **Pour `ProtocolSmith`** : Orchestration des scénarios réseau MSP (clients/serveurs concurrents, injection de latence/chaos).
* **Pour `ouvrage-doc-etl` / `relieur`** : Séquençage des pipelines de conversion (PDF $\rightarrow$ Pandoc AST $\rightarrow$ KCL).
* **Pour l'ERP Data-Centric** : Exécution des gardiens BPMN et validation des transitions d'états d'actes documentaires.

---

## 4. Stack Technologique conseillée

* **CLI & Config** : `cobra` / `viper`.
* **Validation de Schéma** : `cuelang.org/go/cue` (définitions embarquées via `go:embed`).
* **Moteur d'Expressions** : `expr-lang/expr` (évaluation `{{...}}` au runtime et `[[...]]` au build).
* **Automate FSM** : `looplab/fsm` pour la traversée déterministe d'états.

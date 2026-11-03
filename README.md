# V++ (Vibe++) Programming Language

> **A declarative, contract-bounded, bidirectional AI abstraction layer.**

V++ is an experimental language specification and architecture designed to solve the core failure modes of natural language "vibe coding": ambiguity, statelessness, context window bloat, and architectural drift. 

Instead of replacing deterministic code with unstructured prompts, V++ enforces **strict, strongly typed I/O boundaries (Contracts)** while allowing **fluid natural language execution (Logic)** strictly scoped within those boundaries. V++ compiles directly down to deterministic Abstract Syntax Tree (AST) nodes in target languages (defaulting to Python 3.11+ and TypeScript) and maintains bidirectional synchronization via cryptographic hashes in `.vpp.lock` files.

---

## Core Pillars

1. **Contract-Bounded Execution:** Every process is split by a visual floor (`---`). Above the floor is a rigid, strongly typed contract (inputs, outputs, exceptions, invariants); below the floor is high-level declarative logic constrained by the contract.
2. **Deterministic AST Anchoring:** V++ instructions map 1:1 to functional AST nodes in target languages rather than scattered lines of text.
3. **Cryptographic State Parity (`.vpp.lock`):** Tracks SHA-256 hashes of generated AST nodes to enable true round-trip engineering and detect manual code drift.
4. **Tiered Compute Routing:** Native syntax modifiers (`*`, `**`, `***`) route simple transformations to fast, low-cost models and complex algorithmic problems to deep-reasoning models.
5. **Non-Destructive Semantic Linting (The `~` System):** The AI compiler is forbidden from silently refactoring or overwriting downstream dependencies. Breaking changes or structural inefficiencies are flagged using `#~` and proposed inside `~{ ... }` blocks for human review.

---

## Repository Structure & Versioning Workflow

This repository tracks the iterative language design, grammar formalization, and architectural evolution of V++. Each milestone is documented as a complete, self-contained **Master Specification** so that any version can be used directly as an AI System Prompt and developer guide.

### Versioning Strategy
* **`main` branch:** Always holds the latest stable Master Specification (`VPP_MASTER_SPEC.md`) and project overview.
* **Version Branches (`v0.1`, `v0.2`, etc.):** Dedicated branches and tags preserving the exact state of the language specification at each iteration.

---

## Specification Changelog

| Version | Focus Area | Key Features |
| :--- | :--- | :--- |
| **v0.1** | Core Language & Block Mechanics | `@CONFIG`, `@MODEL`, Process Blocks (`---`), ALL CAPS Action Commands (`DO`, `FETCH`, `SAVE`, `EMIT`, `CHECK`, `SET`, `YIELD`), Strictness/Compute Modifiers (`!`, `?`, `*`, `**`, `***`), AI Delegation (`[ ]` / `{ }`), and the Tilde (`~`) Audit System. |
| **v0.2** | Full-Stack & Multi-File Architecture | `DOMAIN` boundaries, declarative `_domain.vpp` manifests, Feature Capsules, the "Rule of One" nesting constraint, Event Entry Points (`ON` clause for HTTP, UI, and Cron), Cross-Domain Public Contracts, declarative Middleware (`-`), and automatic multi-step database transactions (`!`). |

---

## Project Roadmap

- [x] **Phase 1: Core Syntax & Block Specification (v0.1)** — Define contracts, action commands, modifiers, and non-destructive linting.
- [x] **Phase 2: Full-Stack & Multi-File Architecture (v0.2)** — Define domain scoping, event triggers, and cross-boundary communication without traditional import boilerplate.

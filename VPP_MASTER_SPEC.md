# V++ Master Specification & AI System Prompt (Version 0.2)

**Version:** 0.2 (Full-Stack & Multi-File Architectural Specification)  
**Status:** Complete Language & Project Structure Specification  
**Target Compilation:** Deterministic AST Nodes (Python 3.11+ / TypeScript / Full-Stack Runtimes)

This document serves as the canonical specification and operational guide for the V++ programming language workspace. Version 0.2 expands the foundational v0.1 block mechanics to support multi-file, full-stack, and enterprise-scale systems without introducing bloated traditional programming boilerplate.

### What's New in v0.2 (Release Highlights)
* **Domain-Based Environment Scoping:** Organizes multi-file projects into `DOMAIN` boundaries configured by declarative `_domain.vpp` manifests (`Target: Client`, `Target: Server`, etc.) rather than manual file imports.
* **Feature Capsules & The "Rule of One":** Groups workflows into flat `.vpp` Feature Capsules and caps sub-folder nesting inside a Domain to a maximum of 1 level deep to protect AI context limits and AST graph traversal.
* **Event Entry Points (`ON` Clause):** Wires Process Contracts directly to HTTP routes (`ON POST`), UI lifecycles (`ON LOAD`), and background cron jobs (`ON SCHEDULE`).
* **Cross-Domain Public Contracts:** Enables type-safe RPC/network calls across domains via `@DomainName.@ProcessName(...)` using `PUBLIC` declarations in `_domain.vpp`.
* **Zero-Syntax Middleware & Multi-Step Transactions:** Leverages existing `-` contract constraints for declarative middleware/auth interceptors and automatically wraps multiple `!` I/O commands inside atomic database transactions.

---

## 1. System Identity & Core Philosophy
* **Language Name:** V++ (Vibe++)
* **Paradigm:** Declarative, contract-bounded, bidirectional AI abstraction layer.
* **Core Philosophy:** Natural language "vibe coding" scales poorly because it is ambiguous, stateless, and disconnected from the codebase. V++ bridges this gap by enforcing **deterministic, strongly typed I/O boundaries (Contracts)** while providing **fluid natural language execution (Logic)** strictly scoped within those boundaries.
* **Target Output:** Compiles down to standard deterministic code (Default: Python 3.11+ / TypeScript AST nodes) scoped by each Domain's target runtime.
* **Parity & Round-Trip Sync:** Instructions map 1:1 to Abstract Syntax Tree (AST) nodes, not arbitrary lines of text. Parity between high-level V++ and raw generated code is tracked via cryptographic hashes in `.vpp.lock` files.

---

## 2. Project Structure & Environment Scoping (New in v0.2)
To support multi-file, full-stack systems without traditional `import` / `require` boilerplate, V++ organizes code into a shallow hierarchy of **Domains** and **Feature Capsules**.

```text
my_fullstack_project/
├── vpp.json                             <-- Root workspace config & lockfile manager
└── Domains/
    ├── Auth_Service/                    <-- Server Domain (Bounded Service Context)
    │   ├── _domain.vpp                  <-- Declarative Domain Manifest (Zero @Process logic)
    │   ├── session_login.vpp            <-- Feature Capsule
    │   └── token_refresh.vpp            <-- Feature Capsule
    │
    ├── Billing_Service/                 <-- Server Domain
    │   ├── _domain.vpp                  <-- Declarative Domain Manifest
    │   ├── direct_checkout.vpp          <-- Feature Capsule
    │   └── Subscriptions/               <-- Grouping Folder (Max 1 level deep)
    │       ├── recurring_billing.vpp
    │       └── cancellation_flow.vpp
    │
    └── Storefront_Web/                  <-- Client Domain
        ├── _domain.vpp                  <-- Declarative Domain Manifest (Target: Client)
        └── dashboard_view.vpp           <-- Feature Capsule
```

### 2.1 Domains & Bounded Service Contexts
A `DOMAIN` represents an independent architectural or service boundary (e.g., `Storefront_Web`, `Auth_Service`, `Billing_Service`). For massive enterprise workspaces, domains may be visually grouped inside high-level cluster folders, which contain no manifests or code of their own.

### 2.2 The Domain Manifest (`_domain.vpp`)
Every Domain folder must contain a `_domain.vpp` file at its root.
* **Strictly Declarative Rule:** `_domain.vpp` defines the execution target, compute routing, exported public process signatures, and shared domain `@MODEL` shapes. **It must never contain executable `@Process` blocks.**

```v++
# Example: Domains/Billing_Service/_domain.vpp
@DOMAIN Billing_Service
  Target: Server/Python3.11
  PUBLIC: @ProcessChargeCard, @ProcessIssueRefund

@CONFIG
  Default_Compute: *

@MODEL PaymentReceipt
  receipt_id String
  amount Float
  status String
```

### 2.3 Feature Capsules & The "Rule of One"
* **Feature Capsules:** Executable `@Process` blocks live in `.vpp` files organized around complete user stories or workflows (e.g., `direct_checkout.vpp`) rather than technical layers (`controllers/`, `utils/`).
* **The "Rule of One":** Inside a Domain folder, sub-folders can only be nested **a maximum of one level deep** (e.g., `Domains/Billing_Service/Subscriptions/recurring_billing.vpp`). Deeper folder nesting is a syntax/structure violation to prevent prompt token explosion and keep the Tilde (`~`) dependency graph shallow.

---

## 3. Global Directives & Configuration

### 3.1 `@CONFIG`
Defines workspace or domain-level compute routing and token budgets. If a command or prompt does not specify a compute tier, it falls back to `Default_Compute`.

```v++
@CONFIG
  Default_Compute: *
  *   Model: Fast-Model-Flash, Tokens: 2000
  **  Model: Standard-Pro, Tokens: 8000
  *** Model: High-Compute-Reasoning, Tokens: 32000
```

### 3.2 `@MODEL` (Data Shapes)
Data structures passed between Process Blocks (including UI views and API payloads) must be declared explicitly to eliminate hallucinated attributes.

```v++
@MODEL UserObject
  id String
  email String
  role String
  created_at Integer
```

**Primitive & Composite Types Supported:**
* **Primitives:** `String`, `Integer`, `Float`, `Boolean`
* **Collections:** `List<T>`, `Map<KeyType, ValueType>`
* **Custom:** Any declared `@MODEL` identifier (including UI component shapes like `UIView`).

---

## 4. Process Block Anatomy & Entry Points
The fundamental unit of execution is the **Process Block**, divided by a visual floor (`---`) into a **Contract** (above) and **Logic** (below) using 2-space indentation.

```v++
###
@ProcessUserLogin
  ON POST "/api/login"
  (credentials AuthModel) -> SessionToken
  THROWS InvalidCredentials, RateLimitExceeded
  + bcrypt_lib
  - Requires: @Auth_Service.@ProcessCheckRateLimit
  - Limit: execution < 200ms
  * LocalModelOverride
  ---
  SET user = FETCH! database for credentials.email
  CHECK! user.password_hash matches credentials.password
  SET token = DO generate session token for user
  YIELD token
```

### Contract Elements (Above `---`):
1. **Declaration (`@`):** Unique PascalCase identifier within its Domain.
2. **Entry Point (`ON` — New in v0.2):** Optional declarative event trigger that instructs the AI compiler to generate routing, UI bindings, or cron schedulers without server boilerplate:
    * **HTTP / API Routes:** `ON GET "/api/items"`, `ON POST "/api/login"`
    * **Client / UI Events:** `ON LOAD "/dashboard"`, `ON CLICK "submit_btn"`
    * **Scheduled / Cron Jobs:** `ON SCHEDULE "every 1st of month"`
3. **Signature:** `(arg1 Type, ...) -> ReturnType` (The `->` operator is mandatory).
4. **`THROWS` (Optional):** Comma-separated list of expected exceptions.
5. **`+` (Dependencies):** External libraries or packages required.
6. **`-` (Constraints, Invariants & Middleware):** Execution rules, latency bounds, and **declarative middleware** (New in v0.2). Declaring `- Requires: @Auth_Service.@ProcessValidateSession` instructs the AI compiler to automatically wrap the generated endpoint AST node in authentication middleware before executing the block's logic.
7. **`*` (AI Spec Override):** Optional local override for the AI model/tokens.
8. **`---` (Visual Floor):** Marks the end of the contract boundary.

---

## 5. Cross-Domain Communication (New in v0.2)
Domains interact as bounded contexts without manual file imports or boilerplate HTTP client code.

* **Implicit Domain Namespacing:** Within the same Domain, processes call each other directly (`DO @HelperProcess(arg)`). To invoke a process across domains, prefix the call with the target Domain name: `DO @DomainName.@ProcessName(arg)`.
* **Public API Enforcement:** Only processes listed as `PUBLIC` (or `EXPORTED`) in a domain's `_domain.vpp` manifest can be invoked by external domains.
* **Automated Network/RPC Bridging:** When a Client Domain (`Storefront_Web`) invokes a Server Domain (`Billing_Service`), the AI compiler reads the contract signature and automatically generates the type-safe network request (REST/RPC) bridging the two environments.

```v++
# Inside Domains/Storefront_Web/dashboard_view.vpp (Client Domain)
###
@ProcessRenderDashboard
  ON LOAD "/dashboard"
  (user_id String) -> UIView
  ---
  # Automatically compiles into a type-safe client-to-server network request:
  SET billing_history = DO @Billing_Service.@ProcessGetInvoices(user_id)
  SET view = DO build dashboard layout with billing_history
  YIELD view
```

---

## 6. Logic & Action Commands (Below `---`)

### 6.1 The First-Word ALL CAPS Rule
Every executable line within a Process Block **must start with an ALL CAPS action command**. Lowercase pseudocode is strictly prohibited from beginning a line.

### 6.2 State & Assignment
* **`SET` is the only valid way to assign state.**
* Direct variable declarations (e.g., `user = FETCH...`) are **syntax errors**.
* Format: $\text{SET}\ \langle\text{variable\_name}\rangle = \langle\text{COMMAND}\rangle\ \langle\text{natural language / target}\rangle$

### 6.3 Command Reference Table

| Command | Category | Description |
| :--- | :--- | :--- |
| `DO` | Transformation | Data transformation, UI layout building, algorithmic math, or invoking Process Blocks (`DO @OtherProcess(args)`). |
| `FETCH` | External I/O | Read operations from databases, APIs, file systems, or external caches. |
| `SAVE` | External I/O | Mutating or writing operations to databases, storage, or external sinks. |
| `EMIT` | Side-Effect | Asynchronous event publishing, webhooks, or messaging bus queues. |
| `CHECK` | Validation | Boolean assertion of a truth condition. |
| `SET` | State | Assigns the output of an operation to a scoped variable. |
| `YIELD` | Output | Returns the final value from the block, strictly matching the Contract's `ReturnType`. |
| `IF` / `ELSE IF` / `ELSE` | Control Flow | Conditional branching. Scoped using 2-space indentation. |
| `FOR` | Control Flow | Collection iteration. Syntax: `FOR <item> in <collection>`. Scoped using 2-space indentation. |

---

## 7. Command Modifiers & Multi-Step Transactions
Modifiers are attached directly to action commands using the stacking order: `COMMAND[Strictness][Compute]`.

### 7.1 Strictness Modifiers & Database Transactions
* **`!` (Strict / Transactional):** Must succeed. If the operation fails, execution halts, triggers an immediate rollback, or raises a declared `THROWS` exception.
    * **Multi-Step Transaction Rule (New in v0.2):** If a single Process Block contains multiple `!` external I/O operations (e.g., multiple `SAVE!` commands), the AI compiler automatically wraps the entire generated AST function in an atomic database transaction. If any step fails, all prior `!` mutations in that block roll back together.
* **`?` (Optional / Safe-Fail):** Safe execution. Errors are suppressed and resolve safely to `null` or empty collections.

```v++
###
@ProcessTransferFunds
  (from_acc String, to_acc String, amount Float) -> Boolean
  THROWS InsufficientFunds, TransferFailed
  ---
  # Compiler automatically wraps both SAVE! operations in a single atomic DB transaction:
  SET debit = SAVE! database deduct amount from from_acc
  SET credit = SAVE! database add amount to to_acc
  YIELD true
```

### 7.2 Compute Modifiers
* **`*`:** Tier 1 / Low Compute (Default, fast models, simple transformations).
* **`**`:** Tier 2 / Medium Compute (Reasoning models, standard algorithms).
* **`***`:** Tier 3 / High Compute (Architectural planning, complex edge cases).

---

## 8. AI Delegation, Documentation & The Tilde (`~`) System

### 8.1 AI Delegation & Scoping (`[ ]` and `{ }`)
Unwritten routines are delegated using Prompt Brackets `[<stars> <intent> ]` and compiled strictly inside Scope Brackets `{ }` directly below the prompt:

```v++
[** Sanitize input list, remove null values, and sort by created_at descending ]
{
  *# Filters out null records and sorts using standard timestamp comparison
  SET valid_items = DO filter items where item is not null
  SET sorted_items = DO sort valid_items by created_at desc
}
```

### 8.2 Documentation Engine Directives
* **`#` (Sacred Human Comment):** Immutable. The AI compiler is strictly forbidden from editing, moving, or deleting these lines.
* **`*#` (AI-Maintained Comment):** Context-aware explanations generated and automatically updated by the AI compiler.
* **`###` (Auto-Comment Directive):** Placed directly above `@ProcessName` to trigger continuous maintenance of `*#` comments across the block.

### 8.3 Semantic Linter & The Tilde (`~`) Audit System
The AI compiler operates non-destructively. Whether a contract change breaks downstream callers across domains or a developer writes inefficient sequential processes, the AI must never silently rewrite project structure. It flags issues and proposes fixes using the **Tilde (`~`) System**:
* **`~#`:** Audit warning tag appended to a broken or inefficient block.
* **`[~ ... ]`:** Review question explaining the breakage or optimization.
* **`~{ ... }`:** Non-destructive replacement proposal awaiting human approval.

```v++
###
@ProcessCheckout ~# BROKEN: @Billing_Service.@ProcessChargeCard now yields PaymentReceipt (was String)
  ON POST "/api/checkout"
  (cart CartModel) -> Boolean
  ---
  SET receipt = DO @Billing_Service.@ProcessChargeCard(cart)
  
  [~ Review fix: Update SAVE! to persist receipt.receipt_id instead of raw object? ]
  ~{ SET saved = SAVE! receipt.receipt_id to orders_db }
  
  SET saved = SAVE! receipt to orders_db
  YIELD true
```

---

## 9. AI Agent Verification Checklist (v0.2)
When generating, auditing, or refactoring V++ v0.2 code, the AI must verify:
1. Are Domains organized cleanly with a strictly declarative `_domain.vpp` manifest containing zero executable `@Process` blocks?
2. Do sub-folders inside any Domain strictly obey the **"Rule of One"** (maximum 1 level of sub-folder nesting)?
3. Are cross-domain calls properly namespaced (`@Domain.@Process`) and exposed as `PUBLIC` in the target `_domain.vpp`?
4. Are middleware requirements declared via `- Requires:` in the Contract rather than duplicated inside logic blocks?
5. Does every executable line below `---` begin with an ALL CAPS action command, and do all state assignments use `SET <var> = <COMMAND>`?
6. Are all modifiers stacked as `COMMAND[Strictness][Compute]`, and are multi-step `!` mutations compiled as atomic transactions?
7. Are all downstream contract breakages and structural optimizations proposed non-destructively using `~#`, `[~ ... ]`, and `~{ ... }`?
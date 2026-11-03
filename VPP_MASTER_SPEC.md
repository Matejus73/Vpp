# V++ Master Specification & AI System Prompt (Version 0.1)

**Version:** 0.1 (Base Specification)  
**Status:** Core Language & Block-Level Architecture  
**Target Compilation:** Deterministic AST Nodes (Python 3.11+ / TypeScript)

This document serves as the canonical specification and operational directive for the V++ programming language workspace. Any AI operating within this project must strictly adhere to the syntax, scoping models, and execution semantics defined below.

---

## 1. System Identity & Core Philosophy
* **Language Name:** V++ (Vibe++)
* **Paradigm:** Declarative, contract-bounded, bidirectional AI abstraction layer.
* **Core Philosophy:** Natural language "vibe coding" scales poorly because it is ambiguous, stateless, and disconnected from the codebase. V++ bridges this gap by enforcing **deterministic, strongly typed I/O boundaries (Contracts)** while providing **fluid natural language execution (Logic)** strictly scoped within those boundaries.
* **Target Output:** Compiles down to standard deterministic code (Default: Python 3.11+ / TypeScript AST nodes).
* **Parity & Round-Trip Sync:** Instructions map to Abstract Syntax Tree (AST) nodes, not arbitrary lines of text. Parity between high-level V++ and raw generated code is tracked via cryptographic hashes in `.vpp.lock` files.

---

## 2. Global Directives & Configuration

### 2.1 `@CONFIG`
Every project file or workspace root defines global compute routing and token budgets. If a command or prompt does not specify a compute tier, it falls back to `Default_Compute`.

```v++
@CONFIG
  Default_Compute: *
  *   Model: Fast-Model-Flash, Tokens: 2000
  **  Model: Standard-Pro, Tokens: 8000
  *** Model: High-Compute-Reasoning, Tokens: 32000
```

### 2.2 `@MODEL` (Data Shapes)
Data structures passed between Process Blocks must be declared explicitly to eliminate hallucinated attributes.

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
* **Custom:** Any declared `@MODEL` identifier

---

## 3. Process Block Anatomy
The fundamental unit of execution is the **Process Block** (or **Process**).
* **Visual Floor Divider:** `---` separates the **Contract** (above) from the **Logic** (below).
* **Indentation:** 2 spaces.

```v++
###
@ProcessName
  (input_1 Type, input_2 Type) -> OutputType
  THROWS ErrorTypeA, ErrorTypeB
  + dependency_library
  - constraint_invariant
  * LocalModelOverride
  ---
  SET result = DO logic instruction
  YIELD result
```

### Contract Elements (Above `---`):
1. **Declaration (`@`):** Unique PascalCase identifier.
2. **Signature:** `(arg1 Type, ...) -> ReturnType` (The `->` operator is mandatory).
3. **`THROWS` (Optional):** Comma-separated list of expected exceptions.
4. **`+` (Dependencies):** External libraries or packages required.
5. **`-` (Constraints/Invariants):** Performance limits, complexity boundaries, or execution rules (e.g., `- Limit: execution < 200ms`, `- Invariant: no raw passwords in memory`).
6. **`*` (AI Spec Override):** Optional local override for the AI model/tokens.
7. **`---` (Visual Floor):** Marks the end of the contract boundary.

---

## 4. Logic & Action Commands (Below `---`)

### 4.1 The First-Word ALL CAPS Rule
Every executable line within a Process Block **must start with an ALL CAPS action command**. Lowercase pseudocode is strictly prohibited from beginning a line.

### 4.2 State & Assignment
* **`SET` is the only valid way to assign state.**
* Direct variable declarations (e.g., `user = FETCH...`) are **syntax errors**.
* Format: $\text{SET}\ \langle\text{variable\_name}\rangle = \langle\text{COMMAND}\rangle\ \langle\text{natural language / target}\rangle$

```v++
# Correct:
SET user_profile = FETCH! CRM for customer data
SET is_valid = CHECK email matches standard pattern

# Incorrect:
user_profile = FETCH! CRM for customer data
```

### 4.3 Command Reference Table

| Command | Category | Description |
| :--- | :--- | :--- |
| `DO` | Transformation | Data transformation, algorithmic math, or invoking other Process Blocks (`DO @OtherProcess(args)`). |
| `FETCH` | External I/O | Read operations from databases, APIs, file systems, or external caches. |
| `SAVE` | External I/O | Mutating or writing operations to databases, storage, or external sinks. |
| `EMIT` | Side-Effect | Asynchronous event publishing, webhooks, or messaging bus queues. |
| `CHECK` | Validation | Boolean assertion of a truth condition. |
| `SET` | State | Assigns the output of an operation to a scoped variable. |
| `YIELD` | Output | Returns the final value from the block, strictly matching the Contract's `ReturnType`. |
| `IF` / `ELSE IF` / `ELSE` | Control Flow | Conditional branching. Scoped using 2-space indentation. |
| `FOR` | Control Flow | Collection iteration. Syntax: `FOR <item> in <collection>`. Scoped using 2-space indentation. |

---

## 5. Command Modifiers & Order of Operations
Modifiers are attached directly to action commands to define operational strictness and AI reasoning allocation.

### Stacking Rule: `COMMAND[Strictness][Compute]`
Strictness modifiers must **always** precede compute modifiers.

### Strictness Modifiers:
* **`!` (Strict / Transactional):** Must succeed. If the operation fails, execution errors, triggers an immediate rollback, or raises a declared `THROWS` error.
* **`?` (Optional / Safe-Fail):** Safe execution. If the operation fails, it suppresses errors and resolves safely to `null` or empty collections.

### Compute Modifiers:
* **`*`:** Tier 1 / Low Compute (Default, fast models, simple transformations).
* **`**`:** Tier 2 / Medium Compute (Reasoning models, standard algorithms).
* **`***`:** Tier 3 / High Compute (Architectural planning, complex edge cases).

**Examples:**
```v++
SET legacy_doc = FETCH?* cache for document_id
SET record = SAVE!*** sanitized_data to database
CHECK! account.balance >= purchase.price
```

---

## 6. AI Delegation & Scoping (`[ ]` and `{ }`)
When a developer delegates an unwritten routine to the AI, they use Prompt Brackets with a compute tier tag:
* **Format:** `[<stars> <natural language intent> ]`
* **Scoping Rule:** The AI compiles this instruction and encapsulates all generated V++ instructions inside Scope Brackets `{ }` directly underneath the prompt.

```v++
[** Sanitize input list, remove null values, and sort by created_at descending ]
{
  *# Filters out null records and sorts using standard timestamp comparison
  SET valid_items = DO filter items where item is not null
  SET sorted_items = DO sort valid_items by created_at desc
}
```

---

## 7. Documentation Engine Directives
* **`#` (Sacred Human Comment):** The AI background compiler is strictly forbidden from editing, moving, or deleting these lines.
* **`*#` (AI-Maintained Comment):** Generated by the AI to explain its internal logic inside `{ }` or directly above commands. Updated automatically if underlying logic changes.
* **`###` (Auto-Comment Directive):** Placed directly above `@ProcessName`. Directs the background compiler to continuously generate and maintain `*#` comments across the entire block.

---

## 8. Semantic Linter & The Tilde (`~`) Audit System
The AI compiler acts non-destructively. When a contract change creates downstream breakages (the "ripple effect") or structural inefficiencies are detected, the AI must never rewrite code autonomously. It flags and proposes fixes using the **Tilde (`~`) System**:
* **`~#` (Audit Warning Tag):** Appended to a broken or inefficient block's header or line.
* **`[~ ... ]` (Proposed Fix Prompt):** The AI's suggested review question.
* **`~{ ... }` (Proposed Replacement Code):** The non-destructive replacement logic, accepted by the developer via IDE actions (e.g., `Tab` completion).

```v++
@Checkout ~# BROKEN: @ProcessPayment now yields ReceiptObject (was String)
  (cart List) -> success Boolean
  ---
  SET receipt = DO @ProcessPayment(cart)
  
  [~ Review fix: Update SAVE to persist receipt.id instead of raw object? ]
  ~{ SET saved_record = SAVE! receipt.id to database }
  
  SET saved_record = SAVE! receipt to database
```

---

## 9. AI Agent Verification Checklist (v0.1)
When generating or refactoring V++ code, the AI must verify:
1. Does every executable line begin with an ALL CAPS command?
2. Are variable assignments written using `SET <var> = <COMMAND>`?
3. Is loop syntax strictly `FOR <item> in <collection>`?
4. Do all modifiers adhere to `COMMAND[Strictness][Compute]`?
5. Are AI-generated subroutines strictly enclosed in `{ }`?
6. Are breaking contract changes flagged via `~#` and `~{ }` rather than destructive overwrites?
# WHAM Primer  
## A One-Page Guide to Building with MVC’s Static and Dynamic Parts

WHAM is a **code-structuring meta-architecture** that decomposes MVC into explicit **static and dynamic components**, making composition, testing, and automation first-class.

WHAM is not a framework.  
It is a **discipline** for how code is organized and how systems are reasoned about.

---

## 1. Canonical Components

| Component | Role |
|---------|------|
| **WINDOW** | Static Models |
| **ADAPTER** | Dynamic Models |
| **IDENTITY** | Dynamic State |
| **INTENT** | Dynamic Views |
| **INTERFACE** | Static Views |
| **ACTION** | Static Control |
| **MANAGER** | Dynamic Control |
| **Instructions** | Static State (this document) |

---

## 2. Core Definitions (Non-Negotiable)

### WINDOW — Static Models
Declared context frames:
- screens, panels, routes, endpoints, DOM roots  
WINDOWS render state and host event slots.

**WINDOWS do not contain business logic.**

---

### ADAPTER — Dynamic Models
Event adapters that convert:
(event + context + identityRef) → INTENT

**Rules**
- No IO
- No state mutation
- Always return an INTENT

---

### IDENTITY — Dynamic State
The single source of truth for state.

Recommended:
- `entityId` (stable)
- `revisionId` (version/hash)
- immutable state snapshot

**All mutable state lives here.**

---

### INTENT — Dynamic Views
A normalized description of *meaning*:
> “What does this interaction mean in context?”

INTENTS are:
- runtime
- serializable
- replayable
- testable

---

### INTERFACE — Static Views
Capability maps:
capability → ACTION delegate

INTERFACES:
- define affordances
- are static definitions
- are selected dynamically by the MANAGER

---

### ACTION — Static Control
Pure functions:
(state, intent) → { delta, effects[] }

**Rules**
- Deterministic
- No IO
- No mutation

---

### MANAGER — Dynamic Control
The runtime composer:
- binds WINDOWS to ADAPTERS
- routes INTENTS to INTERFACES
- executes ACTIONS
- commits deltas to IDENTITY
- runs EFFECTS and workflows

**Only the MANAGER mutates state or performs IO.**

---

## 3. The Four Mandatory Contracts

1. **Adapters always return INTENTS**
2. **Actions always return `{ delta, effects }`**
3. **Only the MANAGER commits state and runs effects**
4. **Interfaces are capability maps, not just types**

Violating these breaks WHAM.

---

## 4. Canonical Runtime Flow
Event
↓
Adapter
↓
Intent
↓
Manager.route()
↓
Interface.delegate
↓
Action
↓
{ delta, effects }
↓
Manager.commit(delta)
Manager.runEffects(effects)
↓
Window.render(identitySnapshot)

---

## 5. WHAM + TDD: Build Order (Per Capability)

For each capability slice:

1. Define the **INTENT**
2. Write **ACTION tests**
3. Define **INTERFACE** mapping
4. Define **IDENTITY** state + revision rules
5. Implement **MANAGER** routing/commit/effects
6. Add **ADAPTER** (event → intent)
7. Add **WINDOW** (host + render)

Repeat per capability.

---

## 6. Folder Structure (Default)
/windows
/adapters
/identity
/intents
/interfaces
/actions
/manager
/effects
/kinds
/tests

---

## 7. Quick Placement Rules

- Renders structure → **WINDOW**
- Bundles inputs into meaning → **ADAPTER → INTENT**
- Stores or versions state → **IDENTITY**
- Defines affordances → **INTERFACE**
- Transforms state purely → **ACTION**
- Wires, routes, commits, runs IO → **MANAGER**

---

## 8. Mental Shifts (Read This Twice)

- Events do not do work — **they express intent**
- Views are not UI — **they are projections**
- Capabilities are data — **interfaces are maps**
- State changes are values — **deltas, not mutations**
- Effects are declared — **not executed in Actions**
- Composition is runtime logic — **the Manager owns it**

---

## 9. Minimal Agent Checklist

Before finalizing code:

- [ ] Adapters return INTENTS only
- [ ] Actions are pure and return `{delta,effects}`
- [ ] Interfaces are delegate maps
- [ ] State lives only in IDENTITY
- [ ] Manager is the only committer/effect runner
- [ ] Flow matches the canonical runtime diagram

---

**WHAM turns architecture into enforceable structure —  
for humans and machine agents alike.**
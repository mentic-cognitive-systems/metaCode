# The WHAM Architecture: Building Better Compositions with Static and Dynamic Parts

This document defines **WHAM** as a *code-structuring meta-architecture* for humans and machine agents.

WHAM decomposes classic MVC and similar architectures into **static and dynamic parts**, adds explicit composition points, and enforces a clean separation between:
- state,
- intent,
- pure logic,
- side effects,
- and runtime composition.

The goal is **composability, testability, replayability, and clarity**.

---
## 0. Core Components
State itself has Static and Dynamic parts:
- Static Instructions: defined contracts, policies, protocols, guidelines, and code (what you're reading right now)
- Dynamic Instances: the active compositions of those instructions at and in a runtime (the actual running system)

With that in mind WHAM uses the following canonical components to structure both the static and dynamic aspects of a system:

 | Component        | Role            |
 |------------------|-----------------|
W| **WRAPPER**      | Static Models   | (aka FRAME, SURFACE, HOST)
A| **ADAPTER**      | Dynamic Models  |
I| **IDENTITY**     | Static Identity |
I| **INSTANCE**     | Dynamic Identity|
I| **INTENT**       | Dynamic Views   |
I| **INTERFACE**    | Static Views    |
∀| **ACTION**       | Static Control  |
M| **MANAGER**      | Dynamic Control |

---

## 1. Canonical Meanings

### WRAPPER — Static Models
Declared context frames:
- Screens
- Panels
- Routes
- Endpoint definitions
- DOM roots

**WRAPPERS host interaction surfaces but do not contain business logic.**

They render from Instance snapshots and expose event slots.

---

### ADAPTER — Dynamic Models
Adapters convert **raw events + runtime context** into **INTENT**.

**Responsibilities**
- Read wrapper-local inputs (forms, selections)
- Read identity references (entityId, revisionId)
- Normalize raw events into a stable Intent

**Rules**
- No IO
- No state mutation
- No business decisions

---

### IDENTITY — Static Identity
Identity is the **stable reference** that names and addresses state.

An Identity is:
- `entityId` — a stable, immutable key (string, URI, composite key)
- `schema` — the shape contract for the state it names
- `versioning policy` — how revisions are tracked (hash, counter, timestamp)

**Identity defines *what* state is and how to find it. It does not hold live values.**

Identity enables:
- stable addressing across contexts
- schema enforcement
- lookup and resolution

---

### INSTANCE — Dynamic Identity
Instance is the **materialized state** that an Identity points to in a given runtime context.

An Instance holds:
- `identityRef` — back-reference to the Identity it materializes
- `revisionId` — the current version/hash
- `state` — the live value snapshot

**All mutable state lives here.**

Multiple Instances of the same Identity may coexist:
- the committed head
- an optimistic branch (during async workflows)
- a stale cache (in another wrapper)
- an undo stack entry

Instance enables:
- versioning
- replay
- undo/redo
- optimistic concurrency
- auditability
- branching and merging of state

---

### INTENT — Dynamic Views
Intent is a normalized description of *meaning*:
> “What someone meant to do.”

**Typical Intent fields**
- `kind`
- `capability`
- `identityRef`
- `payload`
- `wrapperCtx`
- `meta`

**INTENTS are loggable, replayable, testable artifacts.**

---

### INTERFACE — Static Views
Interfaces define **capability surfaces**.

An Interface is:
capability → Action delegate (or pipeline)

Interfaces are:
- static definitions
- data structures (maps of function references)
- dynamically selected by the Manager

They define **policy affordances**, not UI widgets.

---

### ACTION — Static Control
Actions are **pure functions**.

Signature:
(stateSnapshot, intent) → { delta, effects }

**Rules**
- No IO
- No mutation
- No global reads
- Deterministic

Actions may be composed into pipelines.

---

### MANAGER — Dynamic Control
The Manager is the runtime composer.

It has four internal responsibilities:

#### Binder
- Wrapper events → Adapters
- Adapters → Manager dispatch

#### Router
- Intent → Interface → Action delegate
- Policy- and context-driven

#### Workflow
- Multi-step async orchestration
- Sagas / process managers

#### Effect Runner
- Executes declared effects
- Emits follow-up Intents if needed

**The Manager is the only place where:**
- state is committed
- side effects occur

---

## 2. Non-Negotiable Contracts

These contracts must hold everywhere:

1. **Adapters always return Intents**
2. **Actions always return `{ delta, effects }`**
3. **Only the Manager commits state and runs effects**
4. **Interfaces are capability maps, not type annotations**

---

## 3. Canonical Flow

### Composition Time (Binding)
WRAPPER.eventSlot → ADAPTER
INTENT → INTERFACE (via Manager routing)
INTERFACE → ACTION delegates
EFFECTS → effect handlers

### Runtime (Execution)
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
Manager.commit(delta) → Instance(identity)
Manager.runEffects(effects)
↓
Wrapper.render(instance)

---

## 4. Folder Structure

A default layout for humans and agents:
/wrappers        # Static Models
/adapters       # Dynamic Models
/identity       # State addressing + schema
/instances      # Live state + versioning
/intents        # Intent schemas
/interfaces     # Capability maps
/actions        # Pure logic
/manager        # Binding / routing / workflow
/effects        # IO executors
/kinds          # Policy + affordances
/tests         # Unit + integration tests

---

## 5. Type Contracts (Language-Agnostic)

### Intent
{
    kind,
    capability,
    identityRef?,
    payload,
    wrapperCtx,
    meta
}

### Action
(state, intent) → {
    delta,
    effects[]
}

### Interface
{
    delegates: {
        capability: Action | ActionPipeline
    }
}

### Instance
{
    identityRef,
    revisionId,
    state
}

### Manager
dispatch(intent)
bind(wrapper)
commit(delta) → Instance
runEffects(effects)

---

## 6. Composition Patterns

### Capability Overlays
Interfaces compose via layers:
- Base
- Role-based
- Feature-flag
- Experiment / A/B

### Action Pipelines
Pure action chains:
- validate → compute → transform → emit effects

### Event Sourcing
Store Intents or resulting events to:
- replay
- audit
- undo/redo

---

## 7. Rules for Humans

### Code Review Checklist
- Is code in the correct layer?
- Are Actions pure?
- Are effects declared, not executed?
- Is mutable state only in Instance?
- Is Identity just a stable reference (no live values)?
- Is routing centralized in Manager?
- Are Interfaces capability maps?

### “Where does this code go?”
- Renders UI / declares structure → **Wrapper**
- Bundles inputs → **Adapter**
- Describes meaning → **Intent**
- Names/addresses state → **Identity**
- Holds live state → **Instance**
- Defines affordances → **Interface**
- Transforms state → **Action**
- Wires everything → **Manager**

---

## 8. Rules for Machine Agents

### Constraints
1. Never mutate state outside Instance
2. Never perform IO outside Effects
3. Identity is immutable — only Instance holds live state
4. Always introduce:
   - Intent
   - Action
   - Interface mapping
   - Adapter
   - Manager routing
5. Prefer **new capabilities** over conditionals

### Test Generation
- Unit tests for Actions
- Adapter → Intent tests
- Manager routing tests
- Effect execution tests

---

## 9. ASCII Diagram

### Binding
WRAPPER → ADAPTER → MANAGER → INTERFACE → ACTION

### Runtime
Event → Adapter → Intent → Manager → Action → {Δ, Fx}
↓                ↓
Instance.commit    EffectRunner
↓
Wrapper.render(instance)

---

## 10. Core Mental Shifts

1. Events don’t do work — **they express intent**
2. Interfaces are **data**, not just types
3. State changes are **values**, not mutations
4. Effects are **declared**, not executed
5. Composition is **runtime responsibility**
6. Identity is **stable** — Instance is **versioned**, not overwritten

---

**WHAM is not a framework.**  
It is a **thinking tool** that turns architecture into enforceable structure for both humans and machine agents.

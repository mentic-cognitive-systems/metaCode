# The WHAM Architecture: Building Better with MVC's Static and Dynamic Parts

This document defines **WHAM** as a *code-structuring meta-architecture* for humans and machine agents to use as they codify their cognitive creations.

WHAM decomposes MVC into **static and dynamic parts**, adds explicit composition points, and enforces a clean separation between:
- state,
- intent,
- pure logic,
- side effects,
- and runtime composition.

The goal is **composability, testability, replayability, and clarity**.

---

## 0. Core Components

WHAM uses the following canonical components:

| Component  | Role |
|-----------|------|
| **WINDOW** | Static Models |
| **ADAPTER** | Dynamic Models |
| **IDENTITY** | Dynamic State |
| **INTENT** | Dynamic Views |
| **INTERFACE** | Static Views |
| **ACTION** | Static Control |
| **MANAGER** | Dynamic Control |
| **Instructions** | Static State (this guide & contracts) |

The instructions as structured are the Static State. So with that you have complete mental modal for structuring your 
State - the values you want to keep
Models - the ways you want to relate those values to eachother
Views - the ways values are presented, interpreted, and afforded
Controllers - the ways you want to control how the values change
---

## 1. Canonical Meanings

### WINDOW — Static Models
Declared context frames:
- Screens
- Panels
- Routes
- Endpoint definitions
- DOM roots

**WINDOWS host interaction surfaces but do not contain business logic.**

They render from Identity snapshots and expose event slots.

---

### ADAPTER — Dynamic Models
Adapters convert **raw events + runtime context** into **INTENT**.

**Responsibilities**
- Read window-local inputs (forms, selections)
- Read identity references (entityId, revisionId)
- Normalize raw events into a stable Intent

**Rules**
- No IO
- No state mutation
- No business decisions

---

### IDENTITY — Dynamic State
Identity is the **single source of truth** for state.

Recommended structure:
- `entityId` (stable)
- `revisionId` (version/hash)
- immutable state snapshot

**All mutable state lives here.**

Identity enables:
- versioning
- replay
- undo/redo
- optimistic concurrency
- auditability

---

### INTENT — Dynamic Views
Intent is a normalized description of *meaning*:
> “What someone meant to do.”

**Typical Intent fields**
- `kind`
- `capability`
- `identityRef`
- `payload`
- `windowCtx`
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
- Window events → Adapters
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
WINDOW.eventSlot → ADAPTER
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
Manager.commit(delta)
Manager.runEffects(effects)
↓
Window.render(identitySnapshot)

---

## 4. Folder Structure

A default layout for humans and agents:
/windows        # Static Models
/adapters       # Dynamic Models
/identity       # State store + versioning
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
    windowCtx,
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

### Manager
dispatch(intent)
bind(window)
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
- Is state only in Identity?
- Is routing centralized in Manager?
- Are Interfaces capability maps?

### “Where does this code go?”
- Renders UI / declares structure → **Window**
- Bundles inputs → **Adapter**
- Describes meaning → **Intent**
- Stores state → **Identity**
- Defines affordances → **Interface**
- Transforms state → **Action**
- Wires everything → **Manager**

---

## 8. Rules for Machine Agents

### Constraints
1. Never mutate state outside Identity
2. Never perform IO outside Effects
3. Always introduce:
   - Intent
   - Action
   - Interface mapping
   - Adapter
   - Manager routing
4. Prefer **new capabilities** over conditionals

### Test Generation
- Unit tests for Actions
- Adapter → Intent tests
- Manager routing tests
- Effect execution tests

---

## 9. ASCII Diagram

### Binding
WINDOW → ADAPTER → MANAGER → INTERFACE → ACTION

### Runtime
Event → Adapter → Intent → Manager → Action → {Δ, Fx}
↓              ↓
Identity.commit   EffectRunner
↓
Window.render

---

## 10. Core Mental Shifts

1. Events don’t do work — **they express intent**
2. Interfaces are **data**, not just types
3. State changes are **values**, not mutations
4. Effects are **declared**, not executed
5. Composition is **runtime responsibility**
6. Identity is **versioned**, not overwritten

---

**WHAM is not a framework.**  
It is a **thinking tool** that turns architecture into enforceable structure for both humans and machine agents.

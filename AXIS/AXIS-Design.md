# AXIS Design Reference

> Use this document when reasoning about **system architecture, component decomposition, and design decisions** under the AXIS model.
> For code generation, folder layout, type contracts, and implementation rules, see **AXIS-Code.md**.

---

## The Grid (Quick Reference)

| Axis | Immutable Essence | Projected Expression |
|------|-------------------|----------------------|
| **Being** | FORM — static generative structure | INSTANCE — dynamic generated state |
| **Identity** | IDENTITY — immutable sameness | TOKEN — temporal identity projections |
| **Seeing** | FACET — defined capability views | INTENT — active interpretations |
| **Doing** | ACTION — pure control logic | MANAGER — contextual orchestration |
| **Impacting** | EFFECT — declared causal descriptions | IMPACT — runtime causal execution |
| **Bridging** | WRAPPER — structural boundaries | ADAPTER — dynamic transformations |

**Reading the grid:**
- **Columns**: Essence defines *what something is*. Expression defines *how it manifests at runtime*.
- **Rows**: Each axis names a fundamental concern. Every axis has exactly one Essence component and one Expression component.
- **The diagonal**: a thing *exists* (Being), *is itself* (Identity), *is perceived* (Seeing), *acts* (Doing), *causes change* (Impacting), and *connects* (Bridging).

---

## Canonical Meanings

### FORM — Immutable Essence of Being

Forms are the **generative definitions** that determine what state *can* be: schemas, types, templates, structural contracts.

A Form defines:
- `shape` — the structure of valid state (fields, types, constraints)
- `defaults` — initial values and factory patterns
- `invariants` — conditions that must always hold

**Forms generate; they do not hold.**

A Form is to an Instance as a class is to an object, a schema is to a record, a template is to a document. But unlike a class, a Form carries no behavior — only structure.

### INSTANCE — Projected Expression of Being

An Instance is the **materialized state** generated from a Form within a runtime context.

An Instance holds:
- `formRef` — back-reference to the Form it was generated from
- `identityRef` — the Identity this Instance belongs to
- `revisionId` — current version/hash
- `state` — the live value snapshot

**All mutable state lives here.**

Multiple Instances of the same Identity may coexist:
- the committed head
- an optimistic branch (during async workflows)
- a stale cache (in another Wrapper)
- an undo stack entry

### IDENTITY — Immutable Essence of Identity

Identity is the **stable reference** that names and addresses state. It is the principle of *sameness across time*.

An Identity is:
- `entityId` — a stable, immutable key (string, URI, composite key)
- `formRef` — which Form governs this entity's shape
- `versioning policy` — how revisions are tracked (hash, counter, timestamp)

**Identity defines *what* something is and how to find it. It does not hold live values.**

### TOKEN — Projected Expression of Identity

A Token is the **temporal projection** of an Identity into a specific runtime context. It is how an Identity appears as it moves through time, sessions, and scopes.

A Token carries:
- `identityRef` — the Identity it represents
- `scope` — where this token is valid (session, request, transaction)
- `expiry` — when it ceases to be authoritative
- `claims` — contextual attributes (permissions, roles, capabilities)

**Tokens are issued, scoped, and revocable. Identities are none of these.**

### FACET — Immutable Essence of Seeing

Facets define **capability surfaces** — the ways something may be viewed or interacted with.

A Facet is:
- `capability` → `Action` delegate (or pipeline)

Facets are:
- static definitions
- data structures (maps of function references)
- dynamically selected by the Manager based on context

**Facets define policy affordances, not UI widgets.** They answer: "given this context, what can be done?"

### INTENT — Projected Expression of Seeing

Intent is a normalized description of *meaning*:
> "What someone meant to do."

An Intent carries:
- `kind` — the type of intent
- `capability` — which Facet capability this targets
- `identityRef` — the entity being acted upon
- `payload` — the data of the intent
- `wrapperCtx` — which Wrapper context originated this
- `meta` — timestamps, correlation IDs, provenance

**Intents are loggable, replayable, testable artifacts.** They are the system's semantic event stream.

### ACTION — Immutable Essence of Doing

Actions are **pure functions**. They transform state and declare effects without performing either.

Signature:
```
(stateSnapshot, intent) → { delta, effects[] }
```

**Rules:**
- No IO
- No mutation
- No global reads
- Deterministic

Actions may be composed into pipelines: validate → compute → transform → declare effects.

### MANAGER — Projected Expression of Doing

The Manager is the **runtime orchestrator**. It is the only component that wires other components together and drives execution.

It has four internal responsibilities:

- **Binder**: Wrapper events → Adapters → Manager dispatch
- **Router**: Intent → Facet → Action delegate (policy- and context-driven)
- **Workflow**: Multi-step async orchestration, sagas, token lifecycle
- **Committer**: Applies deltas to produce new Instances, issues and revokes Tokens

**The Manager is the only place where state transitions and orchestration occur.**

Note: The Manager does *not* execute effects. That belongs to Impact.

### EFFECT — Immutable Essence of Impacting

An Effect is a **declared description of a side effect** — a pure value that describes what *should* happen in the world, without doing it.

An Effect carries:
- `kind` — the type of effect (IO, emit, notify, persist, schedule)
- `target` — what system/resource is affected
- `payload` — the data to carry
- `retryPolicy` — how failures should be handled
- `compensatingEffect` — what to do on rollback

**Effects are data, not execution.** They are produced by Actions, collected by the Manager, and handed to Impact for execution.

### IMPACT — Projected Expression of Impacting

Impact is the **runtime execution of declared Effects** — the point where the system actually touches the outside world.

Impact responsibilities:
- Execute Effects against real targets (network, disk, external APIs)
- Handle retries, timeouts, and failure recovery
- Execute compensating Effects on rollback
- Emit follow-up Intents when an Effect's result requires further processing
- Report outcomes back to the Manager

**Impact is the only place where side effects actually occur.**

### WRAPPER — Immutable Essence of Bridging

Wrappers are **structural boundaries** that host interaction surfaces.

Examples:
- Screens, panels, routes
- API endpoint definitions
- DOM roots, canvas contexts
- Message queue consumers

**Wrappers host interaction surfaces but do not contain business logic.**

### ADAPTER — Projected Expression of Bridging

Adapters convert **raw events + runtime context** into Intents.

**Rules:**
- No IO
- No state mutation
- No business decisions

Adapters are the *only* path from the outside world into the system's Intent stream.

---

## Canonical Flow

### Composition Time (Binding)

```
Wrapper.eventSlot  →  Adapter
Intent             →  Facet       (via Manager routing)
Facet              →  Action      (delegate resolution)
Effect             →  Impact      (handler registration)
```

### Runtime (Execution)

```
Event (from outside world)
  ↓
Adapter (normalizes to Intent)
  ↓
Intent (semantic description of meaning)
  ↓
Manager.route() (selects Facet by context + Token)
  ↓
Facet.delegate (resolves to Action)
  ↓
Action(instance.state, intent) → { delta, effects[] }
  ↓                                    ↓
Manager.commit(delta)              Impact.execute(effects[])
  ↓                                    ↓
Instance(identity) updated         Side effects occur
  ↓                                    ↓
Token refreshed (if needed)        Follow-up Intents emitted
  ↓
Wrapper.render(instance)
```

---

## Composition Patterns

### Capability Overlays
Facets compose via layers:
- Base capabilities
- Role-based capabilities (driven by Token claims)
- Feature-flag capabilities
- Experiment / A/B capabilities

### Action Pipelines
Pure action chains:
```
validate → compute → transform → declare effects
```

### Effect Batching
Effects declared by multiple Actions within a single workflow can be:
- batched for efficiency
- ordered by dependency
- rolled back atomically via compensating Effects

### Event Sourcing
Store Intents or committed deltas to enable:
- replay
- audit
- undo/redo
- temporal queries

### Token-Scoped Views
Different Tokens for the same Identity can yield different Facets:
- Admin Token → full capability Facet
- Read-only Token → restricted Facet
- Expired Token → re-authentication Intent

### Failure Patterns

Every Intent expresses a desired outcome. When the outcome diverges from the desire, the divergence itself is expressible in AXIS — not as an exception, but as a structured flow through the same components.

**Principle: failures are alternate Effects, not exceptions.**

Three failure points in the canonical flow:

#### 1. Action-level failure (validation, business rule violation)

The Action is pure — it returns a delta and effects that *describe* the failure:

```
function transferFunds(state, intent) {
  if (state.balance < intent.payload.amount) {
    return {
      delta: {},
      effects: [{
        kind: "INTENT_REJECTED",
        target: "caller",
        payload: {
          originalIntent: intent,
          reason: "insufficient_funds",
          details: { available: state.balance, requested: intent.payload.amount },
        },
      }],
    };
  }
  // ... happy path
}
```

#### 2. Impact-level failure (IO error, timeout, external system down)

When Impact cannot execute an Effect, it follows the Effect's declared recovery strategy:

```
// The Effect declares its own failure plan:
{
  kind: "HTTP_POST",
  target: "/api/payments",
  payload: { ... },
  retryPolicy: { maxRetries: 3, backoff: "exponential" },
  compensatingEffect: {
    kind: "ROLLBACK_TRANSFER",
    target: "ledger",
    payload: { reverseOf: originalDelta },
  },
}
```

Impact retries per policy, then executes `compensatingEffect` on exhaustion, then emits a follow-up Intent describing the outcome. The follow-up re-enters the system through the Manager.

#### 3. Manager-level failure (unroutable Intent, missing Facet, expired Token)

The Manager produces Effects directly:

```
if (!facet) {
  await runImpact([{
    kind: "INTENT_UNROUTABLE",
    target: "caller",
    payload: { intent, reason: "no_matching_facet" },
  }], ctx);
  return;
}
```

**At every failure point, the system produces *data* (Effects, follow-up Intents) rather than throwing exceptions.** This makes failures loggable, replayable, testable, and composable.

| Concern | Where it lives |
|---------|----------------|
| "Can this operation succeed?" | **Action** — validates and returns a rejection Effect if not |
| "What if IO fails?" | **Effect** — declares retryPolicy + compensatingEffect |
| "What if IO fails after retries?" | **Impact** — executes compensation, emits follow-up Intent |
| "What do we tell the user/caller?" | **Facet** — an error-handling Facet maps failure Intents to notification Actions |
| "What if the Intent can't be routed?" | **Manager** — produces an INTENT_UNROUTABLE Effect |
| "What's the permanent record?" | **Intent stream** — all Intents (successful and failed) are logged |

### Manager Boundaries

Both worked examples use a single Manager. Real systems have multiple bounded contexts, each with its own Manager. The rule:

**Managers never call each other directly. They communicate through Effects and Intents.**

A Manager in one context produces an Effect (e.g., `{ kind: "EMIT_EVENT", target: "order-service" }`). Impact executes it. The receiving context's Adapter normalizes it into a local Intent, and that context's Manager routes it through its own Facets and Actions.

This means:
- Each Manager owns its own Instances — no shared mutable state
- Each Manager has its own Facet registry — capabilities are local
- Cross-context communication is always asynchronous and explicit
- Failure at the boundary is handled by Effect retry/compensation, not distributed transactions

**One Manager per bounded context. One Instance store per Manager. Effects are the only bridges.**

```
[ Context A ]                    [ Context B ]
Manager A                        Manager B
  ↓                                ↑
Action → Effect                  Adapter → Intent
  ↓                                ↑
Impact ──── (queue/API/store) ────→ Wrapper
```

How to decide Manager boundaries:
- Different domain language? Different Manager.
- Different deployment unit? Different Manager.
- Different team? Almost certainly different Manager.
- Same entity viewed differently? Same Manager, different Facets.

---

## Diagrams

### Binding (Composition Time)
```
WRAPPER → ADAPTER → MANAGER → FACET → ACTION → EFFECT → IMPACT
```

### Runtime Flow
```
Event → Adapter → Intent → Manager → Action → { Δ, Effects }
                    ↑          |          ↓            ↓
                 Token      Facet    Instance     Impact
                 (scope)   (route)  (commit Δ)   (execute)
```

### The Essence–Expression Split
```
    Essence (static)         Expression (dynamic)
    ─────────────────        ─────────────────────
    FORM          ←────→     INSTANCE
    IDENTITY      ←────→     TOKEN
    FACET         ←────→     INTENT
    ACTION        ←────→     MANAGER
    EFFECT        ←────→     IMPACT
    WRAPPER       ←────→     ADAPTER
```

---

## Core Mental Shifts

1. Events don't do work — **they express intent**
2. Effects are data — **Impact is execution**
3. State changes are values — **not mutations**
4. Identity is permanent — **Tokens are temporary**
5. Forms generate — **Instances hold**
6. Facets are data — **not type annotations**
7. Composition is **runtime responsibility** (the Manager's job)
8. Side effects have **two phases**: declaration (Effect) and execution (Impact)

---

## Relationship to Existing Patterns

### MVC → AXIS

| MVC | AXIS Essence | AXIS Expression | What changed |
|-----|--------------|-----------------|--------------|
| Model | FORM (shape) | INSTANCE (state) | Model split into definition vs. live values |
| View | FACET (capabilities) | INTENT (meaning) | View split into "what can be seen" vs. "what was meant" |
| Controller | ACTION (logic) | MANAGER (orchestration) | Controller split into pure logic vs. runtime wiring |
| *(none)* | IDENTITY | TOKEN | Identity axis is new — MVC doesn't address it |
| *(none)* | EFFECT | IMPACT | Impacting axis is new — MVC conflates effects into Controller |
| *(none)* | WRAPPER | ADAPTER | Bridging axis is new — MVC assumes a single context |

### Flux / Redux → AXIS

Flux introduced the unidirectional data flow: Action → Dispatcher → Store → View. AXIS preserves this flow but decomposes it further:

- **Flux Action** ≈ AXIS Intent (a description of what happened)
- **Flux Dispatcher** ≈ AXIS Manager (routes intents to logic)
- **Flux Store** ≈ AXIS Instance (holds state)
- **Flux Reducer** ≈ AXIS Action (pure state transformation)

The key difference: Redux thunks and middleware conflate effect declaration with effect execution. AXIS separates them into EFFECT (data) and IMPACT (execution), making side effects testable without mocking middleware.

### CQRS → AXIS

CQRS separates commands (writes) from queries (reads). In AXIS, this separation emerges naturally from the Facet system:

- A **command** is an Intent whose capability maps to a state-mutating Action in a write Facet
- A **query** is an Intent whose capability maps to a read-only Action in a read Facet
- Different Tokens can grant access to different Facets — command access vs. query access

CQRS is not a separate pattern to adopt; it's a Facet configuration.

### Event Sourcing → AXIS

AXIS's Intent stream *is* an event source. Store Intents (or committed deltas) and you get replay, audit, and temporal queries for free. The difference: AXIS names all the other components around the event stream (Forms validate, Actions transform, Effects declare consequences), while Event Sourcing as typically described leaves those as implementation details.

### Events and Handlers

The generic "event → handler" pattern is the most common abstraction in software. AXIS decomposes it into *specific, testable seams*:

```
Generic:     Event  →  Handler
AXIS:        Event  →  Adapter  →  Intent  →  Manager  →  Facet  →  Action
             (raw)     (normalize)  (meaning)   (route)    (select)   (execute)
```

Each step is independently testable. The generic pattern gives you one seam; AXIS gives you five.

### Roles, Responsibilities, and Capabilities

These three concepts are already fully expressed across the existing grid:

| Concept | AXIS component | How |
|---------|----------------|-----|
| **Role** (who) | TOKEN claims | A role is a claim carried by a Token |
| **Capability** (what) | FACET delegate keys | A capability is an entry in a Facet |
| **Responsibility** (how) | ACTION | A responsibility is a pure function |
| **Authorization** (when/where) | MANAGER routing | The Manager selects a Facet based on Token claims |

A role is not its own architectural component — it's a *value* in a Token that drives Facet selection. Adding a ROLE axis would pull apart what the Token→Facet→Action chain already expresses cleanly.

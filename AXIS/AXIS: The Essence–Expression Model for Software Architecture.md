# AXIS: The Essence–Expression Model for Software Architecture

This document defines **AXIS** as a *meta-architectural model* for structuring codified software systems into coherent compositional components that are easy to understand, maintain, and evolve. The ideal is to enable both human and machine agents to reason about, generate, and orchestrate software architectures with clarity and precision.

AXIS structures codifications — structured statements that derive and drive state — along two orthogonal dimensions: **Immutable Essence** and **Projected Expression**; static and dynamic; being and doing; form and function.

This decomposition clearly separates the *core identity and structure* of a system (Essence) from its *runtime behavior and interactions* (Expression), while maintaining explicit mappings between them.

---

## 0. The Grid

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

**In prose:**

Forms define generative structure, realized as Instances of state.
Identity provides immutable sameness, expressed through Tokens across time.
Facets define how something may be seen, while Intents determine how it is interpreted.
Actions define pure control logic, contextualized and executed by Managers.
Effects declare what should happen; Impacts are what actually does.
Wrappers establish structural boundaries, while Adapters enable dynamic transformation between them.

---

## 1. Canonical Meanings

### FORM — Immutable Essence of Being

Forms are the **generative definitions** that determine what state *can* be: schemas, types, templates, structural contracts.

A Form defines:
- `shape` — the structure of valid state (fields, types, constraints)
- `defaults` — initial values and factory patterns
- `invariants` — conditions that must always hold

**Forms generate; they do not hold.**

A Form is to an Instance as a class is to an object, a schema is to a record, a template is to a document. But unlike a class, a Form carries no behavior — only structure.

---

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

---

### IDENTITY — Immutable Essence of Identity

Identity is the **stable reference** that names and addresses state. It is the principle of *sameness across time*.

An Identity is:
- `entityId` — a stable, immutable key (string, URI, composite key)
- `formRef` — which Form governs this entity's shape
- `versioning policy` — how revisions are tracked (hash, counter, timestamp)

**Identity defines *what* something is and how to find it. It does not hold live values.**

Identity enables:
- stable addressing across contexts
- lookup and resolution
- equality testing ("is this the same entity?")

---

### TOKEN — Projected Expression of Identity

A Token is the **temporal projection** of an Identity into a specific runtime context. It is how an Identity appears as it moves through time, sessions, and scopes.

A Token carries:
- `identityRef` — the Identity it represents
- `scope` — where this token is valid (session, request, transaction)
- `expiry` — when it ceases to be authoritative
- `claims` — contextual attributes (permissions, roles, capabilities)

**Tokens are issued, scoped, and revocable. Identities are none of these.**

Examples:
- An auth session token issued for a user Identity
- A revision cursor pointing to a specific Instance version
- A transaction handle during an optimistic workflow
- A cache key scoping an Instance to a particular Wrapper

---

### FACET — Immutable Essence of Seeing

Facets define **capability surfaces** — the ways something may be viewed or interacted with.

A Facet is:
- `capability` → `Action` delegate (or pipeline)

Facets are:
- static definitions
- data structures (maps of function references)
- dynamically selected by the Manager based on context

**Facets define policy affordances, not UI widgets.** They answer: "given this context, what can be done?"

---

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

---

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

---

### MANAGER — Projected Expression of Doing

The Manager is the **runtime orchestrator**. It is the only component that wires other components together and drives execution.

It has four internal responsibilities:

#### Binder
- Wrapper events → Adapters
- Adapters → Manager dispatch

#### Router
- Intent → Facet → Action delegate
- Policy- and context-driven (Facet selection)

#### Workflow
- Multi-step async orchestration
- Sagas / process managers
- Token lifecycle management

#### Committer
- Applies deltas to produce new Instances
- Issues and revokes Tokens

**The Manager is the only place where state transitions and orchestration occur.**

Note: The Manager does *not* execute effects. That belongs to Impact.

---

### EFFECT — Immutable Essence of Impacting

An Effect is a **declared description of a side effect** — a pure value that describes what *should* happen in the world, without doing it.

An Effect carries:
- `kind` — the type of effect (IO, emit, notify, persist, schedule)
- `target` — what system/resource is affected
- `payload` — the data to carry
- `retryPolicy` — how failures should be handled
- `compensatingEffect` — what to do on rollback

**Effects are data, not execution.** They are produced by Actions, collected by the Manager, and handed to Impact for execution.

This separation is the core of testability: you can assert that an Action declared the correct Effects without ever performing IO.

---

### IMPACT — Projected Expression of Impacting

Impact is the **runtime execution of declared Effects** — the point where the system actually touches the outside world.

Impact responsibilities:
- Execute Effects against real targets (network, disk, external APIs)
- Handle retries, timeouts, and failure recovery
- Execute compensating Effects on rollback
- Emit follow-up Intents when an Effect's result requires further processing
- Report outcomes back to the Manager

**Impact is the only place where side effects actually occur.**

Impact is to Effect as Instance is to Form: the runtime realization of a static declaration.

---

### WRAPPER — Immutable Essence of Bridging

Wrappers are **structural boundaries** that host interaction surfaces.

Examples:
- Screens, panels, routes
- API endpoint definitions
- DOM roots, canvas contexts
- Message queue consumers

**Wrappers host interaction surfaces but do not contain business logic.**

They render from Instances and expose event slots that Adapters bind to.

---

### ADAPTER — Projected Expression of Bridging

Adapters convert **raw events + runtime context** into Intents.

**Responsibilities:**
- Read Wrapper-local inputs (forms, selections, request bodies)
- Read Identity/Token references
- Normalize raw events into stable Intents

**Rules:**
- No IO
- No state mutation
- No business decisions

Adapters are the *only* path from the outside world into the system's Intent stream.

---

## 2. Non-Negotiable Contracts

These contracts must hold everywhere:

1. **Adapters always produce Intents** — never raw mutations or direct Action calls
2. **Actions always return `{ delta, effects[] }`** — never perform IO or mutate state
3. **Effects are data** — declared by Actions, never self-executing
4. **Only Impact executes Effects** — the sole site of side effects
5. **Only the Manager commits state** — producing new Instances and issuing Tokens
6. **Facets are capability maps** — not type annotations or UI components
7. **Forms are generative** — they define shape, not hold values
8. **Identity is immutable** — only Tokens project identity into temporal contexts

---

## 3. Canonical Flow

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

## 4. Folder Structure

A default layout for humans and agents:

```
/forms          # Generative structure (schemas, types, templates)
/instances      # Live state + versioning
/identity       # Stable references + addressing
/tokens         # Temporal identity projections
/facets         # Capability maps
/intents        # Intent schemas
/actions        # Pure logic
/manager        # Binding / routing / workflow
/effects        # Effect declarations + schemas
/impacts        # Effect executors + handlers
/wrappers       # Structural boundaries
/adapters       # Event → Intent transformers
/tests          # Unit + integration tests
```

---

## 5. Type Contracts (Language-Agnostic)

### Form
```
{
    shape,
    defaults,
    invariants[]
}
```

### Instance
```
{
    formRef,
    identityRef,
    revisionId,
    state
}
```

### Identity
```
{
    entityId,
    formRef,
    versioningPolicy
}
```

### Token
```
{
    identityRef,
    scope,
    expiry,
    claims
}
```

### Facet
```
{
    delegates: {
        capability: Action | ActionPipeline
    }
}
```

### Intent
```
{
    kind,
    capability,
    identityRef?,
    payload,
    wrapperCtx,
    meta
}
```

### Action
```
(state, intent) → {
    delta,
    effects[]
}
```

### Effect
```
{
    kind,
    target,
    payload,
    retryPolicy?,
    compensatingEffect?
}
```

### Manager
```
dispatch(intent)
bind(wrapper)
commit(delta) → Instance
issueToken(identity, scope) → Token
revokeToken(token)
```

### Impact
```
execute(effects[]) → outcomes[]
compensate(effects[])
```

---

## 6. Composition Patterns

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

There are three failure points in the canonical flow, and each has a distinct AXIS treatment:

#### 1. Action-level failure (validation, business rule violation)

The Action is pure — it doesn't "fail" in the runtime sense. It returns a delta and effects that *describe* the failure:

```
function transferFunds(state, intent) {
  if (state.balance < intent.payload.amount) {
    return {
      delta: {},  // no state change
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

The rejection is an Effect. Impact delivers it. The caller receives a structured rejection, not a thrown error.

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

// Impact's responsibility:
// 1. Attempt execution
// 2. Retry per retryPolicy
// 3. On exhaustion, execute compensatingEffect
// 4. Emit a follow-up Intent describing the outcome:
//    { kind: "impact.failed", payload: { effect, error, compensated: true } }
```

The follow-up Intent re-enters the system through the Manager, which can route it to an error-handling Facet — logging, user notification, dead-letter queuing, or escalation.

#### 3. Manager-level failure (unroutable Intent, missing Facet, expired Token)

The Manager is the only component that can detect structural failures — an Intent that doesn't match any route, a Token that has expired, a Facet that was unregistered. These are handled by producing Effects directly:

```
// In the Manager's dispatch:
if (!facet) {
  await runImpact([{
    kind: "INTENT_UNROUTABLE",
    target: "caller",
    payload: { intent, reason: "no_matching_facet" },
  }], ctx);
  return;
}
```

**The key insight: at every failure point, the system produces *data* (Effects, follow-up Intents) rather than throwing exceptions.** This makes failures:
- **Loggable** — the same Intent stream captures successes and failures
- **Replayable** — retry a failed Intent against fixed state
- **Testable** — assert that an Action returns the correct rejection Effect without performing IO
- **Composable** — error-handling Facets can be layered just like capability Facets

#### Designing for failure in AXIS

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

A Manager in one context produces an Effect (e.g., `{ kind: "EMIT_EVENT", target: "order-service" }`). Impact executes it — placing a message on a queue, calling an API, writing to a shared store. The receiving context's Adapter normalizes it into a local Intent, and that context's Manager routes it through its own Facets and Actions.

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

## 7. Rules for Humans

### Code Review Checklist
- Is code in the correct component?
- Are Actions pure (no IO, no mutation)?
- Are Effects declared as data, not executed inline?
- Is mutable state only in Instances?
- Is Identity immutable (no live values)?
- Are Tokens scoped and expirable?
- Is side-effect execution only in Impact?
- Is orchestration centralized in the Manager?
- Are Facets capability maps (not type annotations)?
- Are Forms purely structural (no behavior)?

### "Where does this code go?"

| If it... | It belongs in... |
|----------|------------------|
| Defines valid state shape | **Form** |
| Holds live mutable state | **Instance** |
| Names/addresses an entity | **Identity** |
| Projects identity into a session/scope | **Token** |
| Maps capabilities to actions | **Facet** |
| Describes what someone meant to do | **Intent** |
| Transforms state (pure logic) | **Action** |
| Orchestrates routing and workflow | **Manager** |
| Declares a side effect | **Effect** |
| Executes a side effect | **Impact** |
| Hosts a UI surface or endpoint | **Wrapper** |
| Converts raw events to Intents | **Adapter** |

---

## 8. Rules for Machine Agents

### Constraints
1. Never mutate state outside Instance
2. Never perform IO outside Impact
3. Identity is immutable — only Tokens project it into runtime contexts
4. Effects are data — never self-executing
5. Always introduce the full chain:
   - Adapter → Intent → Manager → Facet → Action → Effect → Impact
6. Prefer **new capabilities** (new Facet entries) over conditionals
7. Forms define shape — never embed behavior in a Form

### Test Generation
- **Actions**: unit test with snapshot state + intent → assert delta + effects
- **Adapters**: raw event → assert correct Intent produced
- **Manager routing**: intent + token → assert correct Facet selected
- **Effects**: assert Action declares expected Effect data
- **Impact**: integration test with mocked targets → assert side effects executed
- **Tokens**: assert scoping, expiry, and claim propagation

---

## 9. Diagrams

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

## 10. Core Mental Shifts

1. Events don't do work — **they express intent**
2. Effects are data — **Impact is execution**
3. State changes are values — **not mutations**
4. Identity is permanent — **Tokens are temporary**
5. Forms generate — **Instances hold**
6. Facets are data — **not type annotations**
7. Composition is **runtime responsibility** (the Manager's job)
8. Side effects have **two phases**: declaration (Effect) and execution (Impact)

---

## 11. Relationship to Existing Patterns

AXIS doesn't emerge from nothing — it decomposes and restructures ideas present in MVC, Flux, CQRS, Event Sourcing, and the broader event-driven / handler pattern. Here's specifically what's different.

### MVC → AXIS

MVC has three roles. AXIS observes that each role has a static definition and a dynamic manifestation, then splits accordingly:

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

The generic "event → handler" pattern is the most common abstraction in software. AXIS decomposes it into *specific, testable seams* rather than treating it as atomic:

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

Example: "Admin can delete users" means:
- Token carries `claims: { role: "admin" }` (Identity axis)
- Manager selects `adminFacet` when it sees that claim (Doing axis)
- `adminFacet.delegates["user.delete"]` points to the `deleteUser` Action (Seeing axis)
- The Action is pure logic (Doing axis)

No new axis needed. The existing grid composes roles, capabilities, and responsibilities from its fundamental parts.

---

## 12. Non-Goals

AXIS does not prescribe:
- A specific programming language or framework
- UI component structure or rendering strategy
- A particular database or persistence technology
- Deployment topology or infrastructure patterns

AXIS is a **thinking tool** — a structural vocabulary that turns architecture into enforceable, composable, testable layers for both humans and machine agents.

---

## 13. Worked Example: User Profile Editor (Vanilla JS SPA)

This example traces a single feature — editing a user profile — through all 12 AXIS components. Vanilla JS is used deliberately: AXIS is the only architecture, with no framework concepts to confuse the picture.

> **Note on Shadow DOM**: Each Wrapper below uses Shadow DOM for structural encapsulation. The Shadow DOM is a natural Wrapper — it establishes a boundary, hides internals, and exposes a public surface (attributes, events, slots). The *business* capability surface is the Facet, not the DOM surface.

---

### FORM — `forms/userProfile.js`

The generative definition of valid profile state.

```js
// forms/userProfile.js
export const UserProfileForm = {
  shape: {
    displayName: { type: "string", maxLength: 100, required: true },
    email:       { type: "string", pattern: /^[^@]+@[^@]+$/, required: true },
    bio:         { type: "string", maxLength: 500, required: false },
    avatarUrl:   { type: "string", required: false },
  },
  defaults: {
    displayName: "",
    email: "",
    bio: "",
    avatarUrl: null,
  },
  invariants: [
    (state) => state.displayName.length > 0 || "displayName must not be empty",
    (state) => !state.email || /^[^@]+@[^@]+$/.test(state.email) || "invalid email",
  ],
};
```

---

### IDENTITY — `identity/userIdentity.js`

Stable reference. A user identity is just an ID and a pointer to the Form.

```js
// identity/userIdentity.js
export function createUserIdentity(userId) {
  return Object.freeze({
    entityId: userId,            // stable, immutable
    formRef: "UserProfileForm",
    versioningPolicy: "hash",    // revisions tracked by content hash
  });
}
```

---

### INSTANCE — `instances/profileInstance.js`

Live mutable state. Created from a Form, owned by an Identity.

```js
// instances/profileInstance.js
import { UserProfileForm } from "../forms/userProfile.js";

export function createProfileInstance(identity, state = null) {
  const liveState = state ?? { ...UserProfileForm.defaults };
  return {
    formRef: identity.formRef,
    identityRef: identity.entityId,
    revisionId: hash(liveState),
    state: liveState,
  };
}

export function applyDelta(instance, delta) {
  const next = { ...instance.state, ...delta };
  return {
    ...instance,
    state: next,
    revisionId: hash(next),
  };
}
```

---

### TOKEN — `tokens/sessionToken.js`

Temporal projection of identity into a session scope.

```js
// tokens/sessionToken.js
export function issueSessionToken(identity, claims = {}) {
  return {
    identityRef: identity.entityId,
    scope: "session",
    expiry: Date.now() + 30 * 60 * 1000,  // 30 minutes
    claims: {
      canEdit: true,
      role: "owner",
      ...claims,
    },
  };
}

export function isExpired(token) {
  return Date.now() > token.expiry;
}
```

---

### FACET — `facets/profileFacets.js`

Capability maps selected by the Manager based on Token claims.

```js
// facets/profileFacets.js
import { validateAndSave, cancelEdit } from "../actions/profileActions.js";

// Full edit capabilities — selected when token.claims.canEdit === true
export const editableFacet = {
  delegates: {
    save:   validateAndSave,
    cancel: cancelEdit,
  },
};

// Read-only — selected when token is absent or canEdit === false
export const readOnlyFacet = {
  delegates: {
    // no save, no cancel — only viewing is possible
  },
};
```

---

### INTENT — `intents/profileIntents.js`

Semantic descriptions of what the user meant to do.

```js
// intents/profileIntents.js
export function saveProfileIntent(identityRef, payload, wrapperCtx) {
  return Object.freeze({
    kind: "profile.save",
    capability: "save",
    identityRef,
    payload,         // { displayName, email, bio }
    wrapperCtx,      // which Wrapper originated this
    meta: { timestamp: Date.now() },
  });
}

export function cancelEditIntent(identityRef, wrapperCtx) {
  return Object.freeze({
    kind: "profile.cancel",
    capability: "cancel",
    identityRef,
    payload: null,
    wrapperCtx,
    meta: { timestamp: Date.now() },
  });
}
```

---

### ACTION — `actions/profileActions.js`

Pure functions. Transform state, declare effects. No IO.

```js
// actions/profileActions.js
import { UserProfileForm } from "../forms/userProfile.js";

export function validateAndSave(state, intent) {
  const next = { ...state, ...intent.payload };

  // Check Form invariants
  const violations = UserProfileForm.invariants
    .map((inv) => inv(next))
    .filter((r) => r !== true);

  if (violations.length > 0) {
    return {
      delta: { _validationErrors: violations },
      effects: [],  // no side effects on validation failure
    };
  }

  return {
    delta: intent.payload,
    effects: [
      {
        kind: "HTTP_PUT",
        target: `/api/users/${intent.identityRef}/profile`,
        payload: intent.payload,
        retryPolicy: { maxRetries: 2, backoff: "exponential" },
        compensatingEffect: {
          kind: "ROLLBACK_INSTANCE",
          target: intent.identityRef,
          payload: { restoreTo: state },
        },
      },
    ],
  };
}

export function cancelEdit(state, intent) {
  // Reset to last committed state — no effects needed
  return {
    delta: { _resetToCommitted: true },
    effects: [],
  };
}
```

---

### EFFECT — (declared inline by Actions above)

Effects are data, not a runtime concern. The `effects[]` array returned by `validateAndSave` is the Effect. Restated for clarity:

```js
// What an Effect looks like as a standalone value:
const saveEffect = {
  kind: "HTTP_PUT",
  target: "/api/users/u-1234/profile",
  payload: { displayName: "Alice", email: "alice@example.com" },
  retryPolicy: { maxRetries: 2, backoff: "exponential" },
  compensatingEffect: {
    kind: "ROLLBACK_INSTANCE",
    target: "u-1234",
    payload: { restoreTo: previousState },
  },
};
// This is pure data. Nothing has happened yet.
```

---

### IMPACT — `impacts/httpImpact.js`

The sole site of side effects. Executes declared Effects.

```js
// impacts/httpImpact.js
const handlers = {
  HTTP_PUT: async (effect) => {
    const res = await fetch(effect.target, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(effect.payload),
    });
    if (!res.ok) throw new Error(`PUT failed: ${res.status}`);
    return { ok: true, status: res.status };
  },

  ROLLBACK_INSTANCE: async (effect) => {
    // Manager handles this — emit a follow-up Intent
    return {
      ok: true,
      followUpIntent: {
        kind: "instance.restore",
        identityRef: effect.target,
        payload: effect.payload,
      },
    };
  },
};

export async function execute(effects) {
  const outcomes = [];
  for (const effect of effects) {
    const handler = handlers[effect.kind];
    if (!handler) throw new Error(`No impact handler for: ${effect.kind}`);
    try {
      outcomes.push(await handler(effect));
    } catch (err) {
      // If there's a compensating effect, execute it
      if (effect.compensatingEffect) {
        await execute([effect.compensatingEffect]);
      }
      outcomes.push({ ok: false, error: err.message });
    }
  }
  return outcomes;
}
```

---

### WRAPPER — `wrappers/profileEditor.js`

Structural boundary using Shadow DOM. Hosts the UI, exposes event slots.

```js
// wrappers/profileEditor.js
class ProfileEditor extends HTMLElement {
  constructor() {
    super();
    // Shadow DOM = structural encapsulation boundary (Wrapper)
    this.attachShadow({ mode: "open" });
  }

  // Render from an Instance snapshot — Wrapper never reads state directly
  render(instance) {
    const s = instance.state;
    const errors = s._validationErrors || [];

    this.shadowRoot.innerHTML = `
      <form id="profile-form">
        <label>Name
          <input name="displayName" value="${s.displayName}" />
        </label>
        <label>Email
          <input name="email" type="email" value="${s.email}" />
        </label>
        <label>Bio
          <textarea name="bio">${s.bio}</textarea>
        </label>
        ${errors.length ? `<div class="errors">${errors.join(", ")}</div>` : ""}
        <button type="submit">Save</button>
        <button type="button" id="cancel">Cancel</button>
      </form>
    `;
  }

  // Event slots — the Adapter will bind to these
  get eventSlots() {
    return {
      submit: () => this.shadowRoot.querySelector("#profile-form"),
      cancel: () => this.shadowRoot.querySelector("#cancel"),
    };
  }

  // Read form values — Adapter calls this, Wrapper just provides the surface
  readInputs() {
    const form = this.shadowRoot.querySelector("#profile-form");
    return Object.fromEntries(new FormData(form));
  }
}

customElements.define("profile-editor", ProfileEditor);
export { ProfileEditor };
```

---

### ADAPTER — `adapters/profileAdapter.js`

Converts raw DOM events into Intents. No business logic.

```js
// adapters/profileAdapter.js
import { saveProfileIntent, cancelEditIntent } from "../intents/profileIntents.js";

export function bindProfileAdapter(wrapper, identityRef, dispatch) {
  const slots = wrapper.eventSlots;

  slots.submit().addEventListener("submit", (e) => {
    e.preventDefault();
    const payload = wrapper.readInputs();
    dispatch(saveProfileIntent(identityRef, payload, "profile-editor"));
  });

  slots.cancel().addEventListener("click", () => {
    dispatch(cancelEditIntent(identityRef, "profile-editor"));
  });
}
```

---

### MANAGER — `manager/profileManager.js`

The orchestrator. Wires everything. The only place state is committed.

```js
// manager/profileManager.js
import { editableFacet, readOnlyFacet } from "../facets/profileFacets.js";
import { applyDelta } from "../instances/profileInstance.js";
import { isExpired } from "../tokens/sessionToken.js";
import { execute as runImpact } from "../impacts/httpImpact.js";
import { bindProfileAdapter } from "../adapters/profileAdapter.js";

export function createProfileManager(wrapper, identity, token, instance) {
  let currentInstance = instance;
  let currentToken = token;

  // --- Router: select Facet based on Token ---
  function selectFacet() {
    if (!currentToken || isExpired(currentToken) || !currentToken.claims.canEdit) {
      return readOnlyFacet;
    }
    return editableFacet;
  }

  // --- Dispatch: the main loop ---
  async function dispatch(intent) {
    const facet = selectFacet();
    const action = facet.delegates[intent.capability];

    if (!action) {
      console.warn(`No capability "${intent.capability}" in current facet`);
      return;
    }

    // Run pure Action
    const { delta, effects } = action(currentInstance.state, intent);

    // Commit delta → new Instance
    currentInstance = applyDelta(currentInstance, delta);

    // Re-render Wrapper with new Instance
    wrapper.render(currentInstance);

    // Hand Effects to Impact for execution
    if (effects.length > 0) {
      const outcomes = await runImpact(effects);

      // Process follow-up Intents from Impact outcomes
      for (const outcome of outcomes) {
        if (outcome.followUpIntent) {
          await dispatch(outcome.followUpIntent);
        }
      }
    }
  }

  // --- Binder: wire Wrapper → Adapter → dispatch ---
  function bind() {
    wrapper.render(currentInstance);
    bindProfileAdapter(wrapper, identity.entityId, dispatch);
  }

  return { bind, dispatch };
}
```

---

### Putting It Together — `app.js`

```js
// app.js — the entry point
import { createUserIdentity } from "./identity/userIdentity.js";
import { createProfileInstance } from "./instances/profileInstance.js";
import { issueSessionToken } from "./tokens/sessionToken.js";
import { createProfileManager } from "./manager/profileManager.js";

// 1. Identity — stable reference
const identity = createUserIdentity("u-1234");

// 2. Token — temporal projection (from auth system)
const token = issueSessionToken(identity, { canEdit: true, role: "owner" });

// 3. Instance — materialized state (from API or cache)
const instance = createProfileInstance(identity, {
  displayName: "Alice",
  email: "alice@example.com",
  bio: "Software engineer",
  avatarUrl: null,
});

// 4. Wrapper — structural boundary (already in DOM)
const wrapper = document.querySelector("profile-editor");

// 5. Manager — wire and go
const manager = createProfileManager(wrapper, identity, token, instance);
manager.bind();

// The runtime flow is now:
//   User clicks Save
//     → Adapter reads form, creates Intent
//     → Manager routes Intent through Token → Facet → Action
//     → Action validates, returns { delta, effects }
//     → Manager commits delta → new Instance → Wrapper re-renders
//     → Impact executes HTTP_PUT effect
//     → On failure, compensating Effect restores previous state
```

---

### What to notice

1. **Every file maps to exactly one AXIS component.** The folder structure *is* the architecture.
2. **The Shadow DOM is a Wrapper**, not a Facet. It provides structural encapsulation. The Facet operates at the business level (what capabilities are available), not the DOM level.
3. **Actions never touched `fetch`**. They declared an Effect. Impact did the fetching.
4. **The Manager is ~40 lines** — routing, committing, delegating. It doesn't contain business logic or IO.
5. **Token drives Facet selection.** Change `canEdit: false` in the Token and the save button still renders (Wrapper doesn't know), but `dispatch` finds no `save` capability in `readOnlyFacet`. The architecture enforces access control without conditionals in the Action.
6. **Testing is trivial per layer**: call `validateAndSave()` with test data → assert delta and effects. No mocking fetch. No rendering DOM. No wiring anything.

---

## 14. Worked Example: Extensible Web Server (Vanilla Node.js)

This example traces the design of a web server where **new routes and services can be dynamically added at runtime** — without restarting, without modifying core files. Vanilla Node.js (`http` module) is used to keep AXIS as the sole architecture.

The key insight: dynamic extensibility is not a special mechanism — it's just the Manager binding new Identities, Forms, and Facets into an already-running composition. The same AXIS flow handles the first route and the hundredth.

---

### FORM — `forms/serviceForm.js`

The generative definition of what a valid service plugin looks like, and what a route registration looks like.

```js
// forms/serviceForm.js

// Every service must conform to this shape
export const ServiceForm = {
  shape: {
    name:        { type: "string", required: true },
    version:     { type: "string", pattern: /^\d+\.\d+\.\d+$/, required: true },
    routes:      { type: "array", items: "RouteForm", required: true },
    healthCheck: { type: "function", required: false },
  },
  defaults: {
    routes: [],
    healthCheck: null,
  },
  invariants: [
    (s) => s.name.length > 0 || "service name must not be empty",
    (s) => s.routes.length > 0 || "service must declare at least one route",
    (s) => s.routes.every(r => r.path.startsWith("/")) || "all routes must start with /",
  ],
};

// Each route within a service
export const RouteForm = {
  shape: {
    method:     { type: "string", enum: ["GET", "POST", "PUT", "DELETE", "PATCH"], required: true },
    path:       { type: "string", required: true },
    capability: { type: "string", required: true },  // maps to a Facet delegate key
  },
  defaults: {
    method: "GET",
  },
  invariants: [],
};
```

---

### IDENTITY — `identity/serviceIdentity.js`

Each service gets a stable identity. Routes are addressed through their parent service.

```js
// identity/serviceIdentity.js
export function createServiceIdentity(serviceName) {
  return Object.freeze({
    entityId: `service:${serviceName}`,
    formRef: "ServiceForm",
    versioningPolicy: "semver",
  });
}

// Routes are sub-identities — addressed as service:name/route:path
export function createRouteIdentity(serviceIdentity, method, path) {
  return Object.freeze({
    entityId: `${serviceIdentity.entityId}/route:${method}:${path}`,
    formRef: "RouteForm",
    versioningPolicy: "hash",
  });
}
```

---

### INSTANCE — `instances/serverInstance.js`

The live routing table. This is the dynamic state of the entire server — a map of registered services and their resolved routes.

```js
// instances/serverInstance.js
export function createServerInstance() {
  return {
    formRef: "ServerForm",
    identityRef: "server:main",
    revisionId: 0,
    state: {
      services: {},     // { [serviceId]: serviceConfig }
      routeTable: [],   // [{ method, path, serviceId, capability }]
    },
  };
}

export function applyDelta(instance, delta) {
  const next = { ...instance.state };

  if (delta.registerService) {
    const svc = delta.registerService;
    next.services = { ...next.services, [svc.identity.entityId]: svc };
    next.routeTable = [
      ...next.routeTable,
      ...svc.routes.map((r) => ({
        method: r.method,
        path: r.path,
        serviceId: svc.identity.entityId,
        capability: r.capability,
      })),
    ];
  }

  if (delta.unregisterService) {
    const id = delta.unregisterService;
    const { [id]: _, ...rest } = next.services;
    next.services = rest;
    next.routeTable = next.routeTable.filter((r) => r.serviceId !== id);
  }

  return {
    ...instance,
    state: next,
    revisionId: instance.revisionId + 1,
  };
}
```

---

### TOKEN — `tokens/requestToken.js`

Each incoming HTTP request gets a Token representing the caller's identity and permissions in this request scope.

```js
// tokens/requestToken.js
export function issueRequestToken(req) {
  const authHeader = req.headers["authorization"] || "";
  const apiKey = req.headers["x-api-key"] || "";

  return {
    identityRef: extractCallerId(authHeader, apiKey),
    scope: "request",
    expiry: Date.now() + 30_000,   // 30 seconds — request-scoped
    claims: {
      roles: extractRoles(authHeader),
      rateLimit: { remaining: 100, window: 60_000 },
    },
  };
}

function extractCallerId(authHeader, apiKey) {
  // Simplified — real implementation verifies JWT / API key
  if (apiKey) return `apikey:${apiKey.slice(0, 8)}`;
  if (authHeader) return `bearer:${hashToken(authHeader)}`;
  return "anonymous";
}

function extractRoles(authHeader) {
  if (!authHeader) return ["public"];
  // Simplified — real implementation decodes JWT claims
  return ["authenticated"];
}

function hashToken(s) { return s.slice(-8); }
```

---

### FACET — `facets/serviceFacets.js`

Capability maps. Each service registers its own Facet. The management API has a separate Facet for service lifecycle operations.

```js
// facets/serviceFacets.js
import { registerService, unregisterService } from "../actions/managementActions.js";

// Management Facet — for adding/removing services at runtime
export const managementFacet = {
  delegates: {
    "service.register":   registerService,
    "service.unregister": unregisterService,
  },
};

// Service Facets are registered dynamically. This is the registry.
const serviceFacets = {};

export function registerServiceFacet(serviceId, facet) {
  serviceFacets[serviceId] = facet;
}

export function getServiceFacet(serviceId) {
  return serviceFacets[serviceId] || null;
}

// Example: a "greeter" service registers its own Facet
// registerServiceFacet("service:greeter", {
//   delegates: {
//     "greet": greeterAction,
//     "greet.personalized": personalizedGreeterAction,
//   },
// });
```

---

### INTENT — `intents/serverIntents.js`

Normalized descriptions of what a request or management operation means.

```js
// intents/serverIntents.js

// An inbound HTTP request, normalized
export function requestIntent(method, path, body, token, wrapperCtx) {
  return Object.freeze({
    kind: "http.request",
    capability: null,        // resolved by Manager via route table
    identityRef: token.identityRef,
    payload: { method, path, body },
    wrapperCtx,
    meta: { timestamp: Date.now(), tokenScope: token.scope },
  });
}

// Register a new service at runtime
export function registerServiceIntent(serviceConfig, token) {
  return Object.freeze({
    kind: "management.register",
    capability: "service.register",
    identityRef: token.identityRef,
    payload: serviceConfig,
    wrapperCtx: "management",
    meta: { timestamp: Date.now() },
  });
}

// Unregister a service
export function unregisterServiceIntent(serviceId, token) {
  return Object.freeze({
    kind: "management.unregister",
    capability: "service.unregister",
    identityRef: token.identityRef,
    payload: { serviceId },
    wrapperCtx: "management",
    meta: { timestamp: Date.now() },
  });
}
```

---

### ACTION — `actions/managementActions.js`

Pure functions for service lifecycle. Also, the pattern for service-specific actions.

```js
// actions/managementActions.js
import { ServiceForm } from "../forms/serviceForm.js";

export function registerService(state, intent) {
  const config = intent.payload;

  // Validate against Form
  const violations = ServiceForm.invariants
    .map((inv) => inv(config))
    .filter((r) => r !== true);

  if (violations.length > 0) {
    return {
      delta: {},
      effects: [{
        kind: "HTTP_RESPONSE",
        target: "caller",
        payload: { status: 400, body: { errors: violations } },
      }],
    };
  }

  // Check for conflicts
  if (state.services[config.identity.entityId]) {
    return {
      delta: {},
      effects: [{
        kind: "HTTP_RESPONSE",
        target: "caller",
        payload: { status: 409, body: { error: "service already registered" } },
      }],
    };
  }

  return {
    delta: { registerService: config },
    effects: [
      {
        kind: "HTTP_RESPONSE",
        target: "caller",
        payload: { status: 201, body: { registered: config.identity.entityId } },
      },
      {
        kind: "LOG",
        target: "audit",
        payload: { event: "service.registered", serviceId: config.identity.entityId },
      },
    ],
  };
}

export function unregisterService(state, intent) {
  const { serviceId } = intent.payload;

  if (!state.services[serviceId]) {
    return {
      delta: {},
      effects: [{
        kind: "HTTP_RESPONSE",
        target: "caller",
        payload: { status: 404, body: { error: "service not found" } },
      }],
    };
  }

  return {
    delta: { unregisterService: serviceId },
    effects: [
      {
        kind: "HTTP_RESPONSE",
        target: "caller",
        payload: { status: 200, body: { unregistered: serviceId } },
      },
      {
        kind: "LOG",
        target: "audit",
        payload: { event: "service.unregistered", serviceId },
      },
    ],
  };
}
```

```js
// actions/greeterActions.js — example service-specific Action
export function greet(state, intent) {
  return {
    delta: {},
    effects: [{
      kind: "HTTP_RESPONSE",
      target: "caller",
      payload: { status: 200, body: { message: "Hello, world!" } },
    }],
  };
}

export function personalizedGreet(state, intent) {
  const name = intent.payload.body?.name || "stranger";
  return {
    delta: {},
    effects: [{
      kind: "HTTP_RESPONSE",
      target: "caller",
      payload: { status: 200, body: { message: `Hello, ${name}!` } },
    }],
  };
}
```

---

### EFFECT — (declared inline by Actions above)

Restated for clarity — the two primary Effect kinds in this server:

```js
// HTTP_RESPONSE — send a response to the caller
{
  kind: "HTTP_RESPONSE",
  target: "caller",
  payload: { status: 200, body: { message: "Hello!" } },
}

// LOG — emit an audit log entry
{
  kind: "LOG",
  target: "audit",
  payload: { event: "service.registered", serviceId: "service:greeter" },
}
```

---

### IMPACT — `impacts/serverImpact.js`

Executes Effects. The HTTP_RESPONSE handler writes to the actual `res` object. This is the *only* place IO occurs.

```js
// impacts/serverImpact.js
const handlers = {
  HTTP_RESPONSE: async (effect, ctx) => {
    const { res } = ctx;
    res.writeHead(effect.payload.status, { "Content-Type": "application/json" });
    res.end(JSON.stringify(effect.payload.body));
    return { ok: true };
  },

  LOG: async (effect) => {
    // In production, this writes to a log service
    console.log(`[AUDIT] ${JSON.stringify(effect.payload)}`);
    return { ok: true };
  },
};

export async function execute(effects, ctx) {
  const outcomes = [];
  for (const effect of effects) {
    const handler = handlers[effect.kind];
    if (!handler) {
      console.error(`No impact handler for: ${effect.kind}`);
      outcomes.push({ ok: false, error: `unknown effect kind: ${effect.kind}` });
      continue;
    }
    outcomes.push(await handler(effect, ctx));
  }
  return outcomes;
}
```

---

### WRAPPER — `wrappers/httpWrapper.js`

The HTTP server itself — a structural boundary that listens for connections and exposes events towards Adapters. It does not parse, route, or respond.

```js
// wrappers/httpWrapper.js
import { createServer } from "node:http";

export function createHttpWrapper(port) {
  const server = createServer();
  let requestHandler = null;

  // Event slot — the Adapter binds here
  function onRequest(handler) {
    requestHandler = handler;
  }

  server.on("request", (req, res) => {
    if (requestHandler) {
      requestHandler(req, res);
    } else {
      res.writeHead(503);
      res.end("Server not ready");
    }
  });

  function start() {
    return new Promise((resolve) => {
      server.listen(port, () => {
        console.log(`Wrapper listening on :${port}`);
        resolve();
      });
    });
  }

  function stop() {
    return new Promise((resolve) => server.close(resolve));
  }

  return { onRequest, start, stop };
}
```

---

### ADAPTER — `adapters/httpAdapter.js`

Converts raw `(req, res)` into Intents. Reads headers, parses body, issues a Token. No routing, no business logic.

```js
// adapters/httpAdapter.js
import { requestIntent, registerServiceIntent, unregisterServiceIntent }
  from "../intents/serverIntents.js";
import { issueRequestToken } from "../tokens/requestToken.js";

export function bindHttpAdapter(wrapper, dispatch) {
  wrapper.onRequest(async (req, res) => {
    // Parse body for non-GET requests
    const body = await readBody(req);
    const token = issueRequestToken(req);

    // Normalize into Intent — the Adapter doesn't decide what to do with it
    const intent = requestIntent(
      req.method,
      req.url,
      body,
      token,
      "httpWrapper"
    );

    // Dispatch to Manager, passing res as context for Impact
    await dispatch(intent, { res, token });
  });
}

function readBody(req) {
  return new Promise((resolve) => {
    if (req.method === "GET" || req.method === "DELETE") return resolve(null);
    const chunks = [];
    req.on("data", (c) => chunks.push(c));
    req.on("end", () => {
      try { resolve(JSON.parse(Buffer.concat(chunks).toString())); }
      catch { resolve(null); }
    });
  });
}
```

---

### MANAGER — `manager/serverManager.js`

Orchestrates routing, dynamic service registration, and the request lifecycle.

```js
// manager/serverManager.js
import { managementFacet, getServiceFacet, registerServiceFacet }
  from "../facets/serviceFacets.js";
import { createServerInstance, applyDelta } from "../instances/serverInstance.js";
import { execute as runImpact } from "../impacts/serverImpact.js";
import { bindHttpAdapter } from "../adapters/httpAdapter.js";

export function createServerManager(wrapper) {
  let instance = createServerInstance();

  // --- Route resolution ---
  function resolveRoute(method, path) {
    return instance.state.routeTable.find(
      (r) => r.method === method && matchPath(r.path, path)
    );
  }

  function matchPath(pattern, actual) {
    // Simplified — production would handle params like /users/:id
    return pattern === actual;
  }

  // --- Dispatch ---
  async function dispatch(intent, ctx) {
    const { token } = ctx;
    let facet, capability;

    if (intent.kind.startsWith("management.")) {
      // Management Intents route to the management Facet
      facet = managementFacet;
      capability = intent.capability;
    } else if (intent.kind === "http.request") {
      // Resolve route from the live routing table
      const route = resolveRoute(intent.payload.method, intent.payload.path);

      if (!route) {
        return await runImpact([{
          kind: "HTTP_RESPONSE",
          target: "caller",
          payload: { status: 404, body: { error: "not found" } },
        }], ctx);
      }

      facet = getServiceFacet(route.serviceId);
      capability = route.capability;

      if (!facet) {
        return await runImpact([{
          kind: "HTTP_RESPONSE",
          target: "caller",
          payload: { status: 502, body: { error: "service facet not loaded" } },
        }], ctx);
      }
    }

    const action = facet.delegates[capability];
    if (!action) {
      return await runImpact([{
        kind: "HTTP_RESPONSE",
        target: "caller",
        payload: { status: 405, body: { error: `no capability: ${capability}` } },
      }], ctx);
    }

    // Run pure Action
    const { delta, effects } = action(instance.state, intent);

    // Commit delta
    if (Object.keys(delta).length > 0) {
      instance = applyDelta(instance, delta);
    }

    // If we just registered a service, also register its Facet
    if (delta.registerService) {
      const svc = delta.registerService;
      registerServiceFacet(svc.identity.entityId, svc.facet);
    }

    // Hand effects to Impact
    await runImpact(effects, ctx);
  }

  // --- Binder ---
  function bind() {
    bindHttpAdapter(wrapper, dispatch);
  }

  // --- Dynamic registration API (for programmatic use) ---
  function registerServiceDirect(serviceConfig, facet) {
    serviceConfig.facet = facet;
    const { delta, effects } = managementFacet.delegates["service.register"](
      instance.state,
      { payload: serviceConfig }
    );
    if (Object.keys(delta).length > 0) {
      instance = applyDelta(instance, delta);
      registerServiceFacet(serviceConfig.identity.entityId, facet);
    }
    return delta;
  }

  return { bind, dispatch, registerServiceDirect };
}
```

---

### Putting It Together — `server.js`

```js
// server.js — the entry point
import { createHttpWrapper } from "./wrappers/httpWrapper.js";
import { createServerManager } from "./manager/serverManager.js";
import { createServiceIdentity } from "./identity/serviceIdentity.js";
import { greet, personalizedGreet } from "./actions/greeterActions.js";

// 1. Wrapper — structural boundary
const wrapper = createHttpWrapper(3000);

// 2. Manager — orchestrator
const manager = createServerManager(wrapper);
manager.bind();

// 3. Register a service dynamically (programmatic)
manager.registerServiceDirect(
  {
    name: "greeter",
    version: "1.0.0",
    identity: createServiceIdentity("greeter"),
    routes: [
      { method: "GET",  path: "/hello",      capability: "greet" },
      { method: "POST", path: "/hello/name",  capability: "greet.personalized" },
    ],
  },
  {
    delegates: {
      "greet":              greet,
      "greet.personalized": personalizedGreet,
    },
  }
);

// 4. Start listening
await wrapper.start();

// Server is live:
//   GET  /hello       → "Hello, world!"
//   POST /hello/name  → "Hello, Alice!"
//
// To add a new service at runtime, POST to a management endpoint
// or call manager.registerServiceDirect() from a plugin loader.
//
// No restart. No recompile. No core file changes.
// The Manager simply binds a new Identity + Facet into the live Instance.
```

---

### What to notice

1. **Dynamic extensibility is just state.** Registering a service = committing a delta to the server Instance. The same `dispatch → Action → delta → commit` flow handles user requests and service registration.
2. **No framework, no magic.** The routing table is a plain array in Instance state. Adding a route appends to it. Removing a service filters it out. Entirely transparent.
3. **The Wrapper never routes.** The `http.createServer` callback just hands `(req, res)` to the Adapter. All routing logic lives in the Manager, informed by the Instance's route table.
4. **Each service is an Identity + Facet pair.** The service's *existence* is its Identity. Its *capabilities* are its Facet. Its *runtime state* (if any) would be its Instance. The service doesn't need to know about the server, HTTP, or other services.
5. **Impact carries the response context.** The `res` object is passed through as `ctx` — never seen by Actions (pure) or Facets (data). Only Impact's `HTTP_RESPONSE` handler touches it.
6. **Testing remains trivial.** Test `registerService()` with mock state → assert delta contains the new service. Test `greet()` → assert it returns an HTTP_RESPONSE effect. No server, no sockets, no HTTP.
# Using WHAM  
## A Practical Guide to Structuring Systems with MVC’s Static and Dynamic Parts  
*(Revised with EFFECTs and EXECUTORs)*

This document explains **how to use the WHAM Architecture** to design, reason about, and build software systems.

WHAM is not a framework.  
It is a **meta-architecture**: a disciplined way to structure and compose codified cognition so that systems are easier to reason about, test, evolve, and automate — both for human *and* machine agents. WHAM's object structure is a consequence of how it guides the structured decomposition thinking of development and compositional execution of software delivering a service via sequential and explicit, testable, composable decisions and actions which  achieve the fulfillment of intended goals.

This guide focuses on:
- how to *think* in WHAM,
- how to *decompose a system* using WHAM,
- and how to *grow a system* using a capability-first, TDD-friendly workflow.

---

## 1. What WHAM Is (and Is Not)

WHAM is an explicit decomposition of MVC into **static and dynamic parts**, plus a small number of additional primitives that make *composition* and *change* first-class concepts.

WHAM does **not** replace MVC.  
It **completes** it.

Traditional MVC collapses many different concerns into three buckets:
- Model
- View
- Controller

WHAM instead asks:

> Which parts of the system are *definitions* and which are *runtime behavior*?  
> Which parts describe *possibility* and which describe *meaning*?  
> Which parts *describe change* and which parts *execute change*?

From those questions, WHAM emerges naturally.

---

## 2. The Canonical Components

WHAM uses the following **core components**:

| Component | Role |
|---------|------|
| **WINDOW** | Static Models |
| **ADAPTER** | Dynamic Models |
| **IDENTITY** | Dynamic State |
| **INTENT** | Dynamic Views |
| **INTERFACE** | Static Views |
| **ACTION** | Static Control |
| **MANAGER** | Dynamic Control |
| **Instructions** | Static State (this guide & contracts) |

### Control Subcomponents (Refinement)

WHAM also makes **change itself** explicit using two **control subcomponents**:

| Subcomponent | Role                        |
|--------------|-----------------------------|
| **EFFECT**   | Static Change Specification |
| **EXECUTOR** | Dynamic Change Interpreter  |

These do **not** replace existing components — they clarify a boundary that already exists in well-designed systems.

---

## 3. The Four Architectural Axes (Now Complete)

WHAM fully specifies the classical architectural axes:

### State
- **IDENTITY** — authoritative, versioned state

### Models
- **WINDOW** — static contextual structure
- **ADAPTER** — dynamic modeling of runtime events

### Views
- **INTERFACE** — static projection of *what is possible*
- **INTENT** — dynamic projection of *what is meant*

### Control
- **ACTION** — static definition of *how state may change*
- **EFFECT** — static definition of *what external change should occur*
- **MANAGER** — dynamic decision of *when and why*
- **EXECUTOR** — dynamic execution of *how the change actually happens*

This is WHAM’s conceptual completeness:  
every meaningful concern in a system has an explicit home.

---

## 4. The Most Important Insight: What a “View” Really Is

In WHAM:

> **A View is not a UI.**  
> **A View is a projection.**

A View answers:
> *How is state being seen, interpreted, or afforded?*

WHAM splits “View” into two fundamentally different jobs.

---

### INTERFACE — Static View (Affordance)

An **INTERFACE** answers:

> *For a thing of this kind, what actions are possible at all?*

Formally:
INTERFACE = capability → ACTION delegate (or pipeline)

Properties:
- static
- declarative
- no state
- no execution

INTERFACE is the entity’s **“in-to-enter face”** — the affordances it presents to the world.

**Why this matters**
- Capabilities become explicit and inspectable
- Policy overlays become composable
- Kinds become flexible instead of rigid types

---

### INTENT — Dynamic View (Declared Meaning)

An **INTENT** is a **runtime-produced, declarative statement of meaning**.

> Given an event, in context, for an identity — what outcome is the user/system *requesting*?

**INTENT is “meaning”, not “method.”**
- ✅ Good INTENT: outcome + parameters + context needed for routing/policy
- ❌ Bad INTENT: steps, function calls, handler names, “first do X then Y then Z”

INTENT:
- is produced at runtime (typically by an ADAPTER)
- is **declarative data** (no executable logic)
- is serializable, replayable, and testable
- is suitable for routing and policy decisions

**INTENT is not a plan.**
It should *not* say “do X, Y, Z”.
Those steps belong to the **MANAGER** (routing/orchestration) and to **ACTION/EFFECT** (state change + external change specification).

**Why this matters**
- Events stop being overloaded with semantics
- Replay, audit, and simulation become natural
- Routing and policy move out of handlers
#### Example

Event: "User clicked checkout button"; this has low meaning; all we know is they clicked a button.
- `Click(ButtonA)`

Possible INTENT (declared meaning; outcome oriented): "User intends to StartCheckout (with these parameters, in this context)"
- `StartCheckout({ cartId, userId, source: "buttonA" })`
- From the action the system may need many steps—but those steps do **not** belong in INTENT; but we can send the intent with the contextual values to the MANAGER for it to decide what to do about it.

Then the MANAGER decides the “how” (declared POLICY (a routing PLAN of ACTIONs and EFFECTs)):
- route `StartCheckout` to an INTERFACE capability
- run ACTION(s) to compute `{ delta, effects[] }`
- commit state and dispatch EFFECT(s) to EXECUTOR(s)

The ACTION(s) compute the state change `{ delta, effects[] }` needed to fulfill the INTENT.

Then the EXECUTOR(s) perform the EFFECT(s) (external changes).


#### Rule of thumb
If an INTENT reads like “call these functions” or “perform these steps”, it’s too procedural; that's the MANAGER’s job. Rewrite it as “the user/system is trying to achieve <outcome> with <parameters>”.
---

## 5. WINDOW Is a Frame for State, Not a View of State
A **WINDOW** is a **frame of observation** serving IDENTITY state.:
- where interaction occurs
- where state is rendered
- where events originate

WINDOWS:
- serve IDENTITIES
- host INTERFACES
- host ADAPTER bindings
- render IDENTITY snapshots

WINDOW is *where* views are applied — not the view itself.

---

## 6. Change Is Two Different Things (Critical Refinement)

WHAM distinguishes **two kinds of change**:

### Static Change — EFFECT
An **EFFECT** is a *declarative specification* of an external change.

Examples:
- HTTP request
- persistence operation
- log emission
- message publication

EFFECTS:
- are inert data
- are serializable
- are replayable
- do not perform the change

---

### Dynamic Change — EXECUTOR
An **EXECUTOR** interprets EFFECTs and performs them in the real world.

EXECUTORS:
- perform IO
- interact with runtimes
- may emit follow-up INTENTs
- never mutate IDENTITY directly

This distinction makes side effects explicit, testable, and auditable.

---
## 7 PLAN vs POLICY (How declaratives derive imperatives)

WHAM works best when you make one additional distinction explicit:

- **PLAN (static)**: a declarative composition describing *which ACTION(s)* and *which EFFECT(s)* are used to fulfill an INTENT.  
  Think: “the recipe card” (inspectable, testable, composable).
- **POLICY (dynamic)**: the runtime decision rules for selecting a PLAN (or a variant) given context.  
  Think: “which recipe applies today, for this user, under these constraints?”

**Where they live**
- PLAN is typically expressed via **INTERFACE capability mappings** and the **ACTION/EFFECT pipeline** they imply.
- POLICY lives in the **MANAGER**: routing, orchestration, guardrails, feature flags, entitlements, A/B, fallback.

**Boundary rule**
- INTENT declares *what is meant*.
- PLAN declares *how the system is designed to fulfill it* (still declarative).
- MANAGER applies POLICY to choose/parameterize the PLAN.
- EXECUTOR performs imperatives by interpreting EFFECTs.


---

## 8. The Non-Negotiable Contracts
WHAM works only if these contracts are enforced in the designs of the logical components:

1. **Adapters always return INTENTs**
   - No IO
   - No state mutation
  
2. **Actions are pure**
    Always return the changed state and the effects created by that change.
    (state, intent) → { delta, effects[] }
   - No IO
   - No mutation
   - No global reads
   - Deterministic

3. **Only the MANAGER commits state**
4. **All external change is described as EFFECT**
5. **Only EXECUTORS perform external change**
6. **INTERFACEs are runtime capability maps**

These are architecturally invariant laws, not conventions or stylistic preferences.

---
## 9. Why Start from the MANAGER’s Perspective

To guide both humans and machine agents, WHAM encourages a **Manager-first mindset**.

Why?

Because the Manager is where:
- GOALs to direct INTENT derive the declarative capabilities which direct the imperative PLANs of ACTION and EFFECT execution
- POLICY becomes PLAN routing
- composition becomes explicit

The Manager asks:
> *What is this system trying to accomplish, and how do interactions flow to make that happen?*

This is the same mental position you take when writing good tests.


## 8. The Canonical Runtime Flow (Final Form)
Event
→ ADAPTER
→ INTENT : Declare the meaning  (outcome + parameters + context): User intends to do X with these values in this context.
→ MANAGER.manage(INTENT): apply POLICY to select/parameterize a PLAN; Given intent X, carry out actions [...]
→ INTERFACE.delegate: PLAN entrypoint:
→ ACTION(s): pure function/transform(s): (state, intent) → { delta, effects }
→ MANAGER.commit(delta): change control awareness and state security
→ MANAGER.dispatchEffects(effects): 
→ EXECUTOR.run(effect): perform external change on behalf of MANAGER with other state MANAGERs
→ (optional) follow-up INTENT
→ WINDOW.render(identitySnapshot)

This flow cleanly separates:
- meaning (INTENT)
- decision (MANAGER/POLICY)
- state change (ACTION + state change commits)
- external execution (EFFECT  → EXECUTOR)

---

## 9. WHAM + Goal-Driven Software Development Process (GDP)  + Test Driven Development (TDD): The Compounding Capability Compiler

WHAM pairs naturally with Goal-Driven Software Development Process (GDP) and Test-Driven Development. It gets you from GOALs, as your top-level organizing principle, driving a clean, composable architecture of INTENTS, PLANs, ACTIONs, and EFFECTs.

For **each capability slice**:

1. Define the GOAL you want to achieve with this capability.
2. Define the **INTENT** (meaning: outcome + parameters).
3. Define the **PLAN** (static composition of ACTION(s) and EFFECT(s) that can fulfill the intent).
4. Define the STATE necessary to support the actions (**IDENTITY** TYPE schema (what it be) + KIND interface (what it do) + versioning).
5. Define the EFFECT shapes the ACTIONs will emit.
6.  Define **INTERFACE** mapping (capability → PLAN entrypoint).
7.  Define **IDENTITY** unique handle + type schema + interface KINDs + versioning
8.  Implement **MANAGER** routing/commit/effect dispatch (**POLICY** lives here).
9.  Implement **EXECUTORS** (imperative interpreters of EFFECT applied to other state MANAGERs).
10. Implement **ADAPTER** (event → intent).
11. Implement **WINDOW** (render + event source).
12. Write **ACTION tests** (pure + deterministic).
13. Add MANAGER tests for policy/routing where needed.

Repeat per capability.

This produces thin, composable vertical slices.

---

## 10. Recursion Without Chaos

WHAM supports recursive composition:
- managers can manage sub-windows
- managers can orchestrate workflows
- executors can emit follow-up intents

Constraints:
- each state domain has a single commit authority
- sub-managers emit intents rather than mutating state directly

---

## 11. Placement Checklist

Ask:

- Is this declaring structure? → WINDOW
- Is this interpreting an event? → ADAPTER → INTENT
- Is this storing/versioning state? → IDENTITY
- Is this defining affordances? → INTERFACE
- Is this transforming state purely? → ACTION
- Is this deciding routing or orchestration? → MANAGER
- Is this describing external change? → EFFECT
- Is this performing external change? → EXECUTOR

If it doesn’t fit cleanly, it’s probably misplaced.

---

## 12. Why WHAM Matters

When used consistently, WHAM gives you:

- explicit architecture instead of implicit coupling
- deterministic, testable core logic
- replayable and auditable behavior
- clean separation of meaning, policy, and execution
- systems that humans and machines can reason about safely

---

## 13. Going Meta: The Static State of Code itself

Code itself is **Static State** that has a purpose of serving the intended goal of it's developers:
- it defines invariants
- it constrains behavior
- it enables enforcement
- 
WHAM helps you structure your code around your goals so that it's structure clearly and explicitly serves those goals.
---

**WHAM is not about writing more code.**  
It is about making *meaning*, *change*, and *control* explicit —  
so complex systems remain understandable as they grow.
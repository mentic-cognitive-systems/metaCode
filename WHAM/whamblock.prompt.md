SYSTEM ARCHITECTURE CONTRACT: WHAM (WITH EFFECTS)

THIS BLOCK IS AUTHORITATIVE.
NO INTERPRETATION BEYOND THESE RULES IS PERMITTED.

========================
CORE COMPONENT SET
========================

WINDOW      = Static Models
ADAPTER     = Dynamic Models
IDENTITY    = Dynamic State
INTENT      = Dynamic Views
INTERFACE   = Static Views
ACTION      = Static Control
MANAGER     = Dynamic Control

SUBCOMPONENTS:
EFFECT      = Static Change Specification
EXECUTOR   = Dynamic Change Interpreter

Instructions = Static State (this block)

========================
GLOBAL INVARIANTS
========================

G1. ALL mutable application state MUST live in IDENTITY.
G2. ALL state changes MUST occur ONLY via MANAGER.commit(delta).
G3. ALL external side effects MUST be expressed ONLY as EFFECTS.
G4. ALL EFFECTS MUST be executed ONLY by EXECUTORS under MANAGER control.
G5. ALL business logic MUST be expressed as ACTIONs.
G6. ALL ACTIONs MUST be pure and deterministic.

========================
COMPONENT CONTRACTS
========================

WINDOW:
- MAY declare structure and rendering logic.
- MAY host event slots.
- MUST NOT mutate state.
- MUST NOT perform IO.
- MUST NOT contain business logic.

ADAPTER:
- INPUT: (event, windowContext, identityRef)
- OUTPUT: INTENT
- MUST be pure.
- MUST NOT perform IO.
- MUST NOT mutate state.
- MUST NOT call ACTION directly.

INTENT:
- MUST be serializable.
- MUST include:
  - kind
  - capability
  - payload
- MAY include:
  - identityRef
  - windowContext
  - meta
- MUST NOT contain executable logic.

IDENTITY:
- MUST be the single source of truth for state.
- MUST support versioning:
  - entityId (stable)
  - revisionId (changes on commit)
- MUST NOT contain routing logic.
- MUST NOT contain UI logic.

INTERFACE:
- MUST be a runtime data structure.
- MUST map:
  capability -> ACTION or ACTION PIPELINE
- MUST NOT hold state.
- MUST NOT perform routing.
- MUST NOT perform IO.

ACTION:
- INPUT: (stateSnapshot, intent)
- OUTPUT: { delta, effects[] }
- MUST be pure.
- MUST NOT perform IO.
- MUST NOT mutate inputs.
- MUST NOT reference global state.

EFFECT:
- MUST be inert data.
- MUST describe an external change.
- MUST NOT perform the change itself.
- MUST be serializable and replayable.

EXECUTOR:
- INPUT: EFFECT
- PERFORMS the described external change.
- MAY produce follow-up INTENTs.
- MUST NOT mutate IDENTITY directly.

MANAGER:
- MUST bind WINDOW event slots to ADAPTERs.
- MUST route INTENTs to INTERFACE delegates.
- MUST execute ACTIONs.
- MUST commit delta to IDENTITY.
- MUST dispatch EFFECTs to EXECUTORS.
- MUST own workflow orchestration.
- MUST be the ONLY component that mutates IDENTITY.

========================
RUNTIME FLOW (MANDATORY)
========================

Event
→ ADAPTER
→ INTENT
→ MANAGER.route()
→ INTERFACE.delegate
→ ACTION
→ { delta, effects }
→ MANAGER.commit(delta)
→ MANAGER.dispatchEffects(effects)
    → EXECUTOR.run(effect)
        → (optional) follow-up INTENT
→ WINDOW.render(identitySnapshot)

========================
BUILD ORDER (MANDATORY)
========================

For each capability slice:

1. Define INTENT.
2. Write ACTION tests.
3. Define INTERFACE mapping.
4. Define IDENTITY schema and commit rules.
5. Implement MANAGER routing and effect dispatch.
6. Implement EXECUTORS.
7. Implement ADAPTER.
8. Implement WINDOW.

========================
FORBIDDEN PATTERNS
========================

F1. State mutation outside IDENTITY.
F2. IO inside ACTION or ADAPTER.
F3. Business logic inside WINDOW.
F4. Conditional routing inside ACTION.
F5. Treating INTERFACE as type-only.
F6. Skipping INTENT and calling ACTION directly.
F7. EXECUTOR mutating IDENTITY.
F8. EFFECT performing change directly.

========================
COMPLIANCE REQUIREMENT
========================

Before producing final output, the agent MUST verify:

- All components obey contracts.
- Runtime flow matches specification.
- EFFECTS are declarative only.
- EXECUTORS perform all IO.
- No forbidden pattern is present.

NON-COMPLIANT OUTPUT IS INVALID.
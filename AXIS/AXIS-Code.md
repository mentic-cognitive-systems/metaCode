# AXIS Code Reference

> Use this document when **generating, reviewing, or refactoring code** under the AXIS model.
> For architectural reasoning, design decisions, and pattern comparisons, see **AXIS-Design.md**.

---

## The Grid (Quick Reference)

| Axis | Essence | Expression |
|------|---------|------------|
| Being | FORM | INSTANCE |
| Identity | IDENTITY | TOKEN |
| Seeing | FACET | INTENT |
| Doing | ACTION | MANAGER |
| Impacting | EFFECT | IMPACT |
| Bridging | WRAPPER | ADAPTER |

---

## Non-Negotiable Contracts

1. **Adapters always produce Intents** — never raw mutations or direct Action calls
2. **Actions always return `{ delta, effects[] }`** — never perform IO or mutate state
3. **Effects are data** — declared by Actions, never self-executing
4. **Only Impact executes Effects** — the sole site of side effects
5. **Only the Manager commits state** — producing new Instances and issuing Tokens
6. **Facets are capability maps** — not type annotations or UI components
7. **Forms are generative** — they define shape, not hold values
8. **Identity is immutable** — only Tokens project identity into temporal contexts

---

## Folder Structure

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

## Type Contracts (Language-Agnostic)

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

## "Where Does This Code Go?"

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

## Rules for Machine Agents

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

## Worked Example: User Profile Editor (Vanilla JS SPA)

This traces a single feature — editing a user profile — through all 12 AXIS components. Every file maps to exactly one component.

> **Shadow DOM is a Wrapper** (structural encapsulation), not a Facet (business capability surface).

### FORM — `forms/userProfile.js`

```js
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

### IDENTITY — `identity/userIdentity.js`

```js
export function createUserIdentity(userId) {
  return Object.freeze({
    entityId: userId,
    formRef: "UserProfileForm",
    versioningPolicy: "hash",
  });
}
```

### INSTANCE — `instances/profileInstance.js`

```js
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

### TOKEN — `tokens/sessionToken.js`

```js
export function issueSessionToken(identity, claims = {}) {
  return {
    identityRef: identity.entityId,
    scope: "session",
    expiry: Date.now() + 30 * 60 * 1000,
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

### FACET — `facets/profileFacets.js`

```js
import { validateAndSave, cancelEdit } from "../actions/profileActions.js";

export const editableFacet = {
  delegates: {
    save:   validateAndSave,
    cancel: cancelEdit,
  },
};

export const readOnlyFacet = {
  delegates: {},
};
```

### INTENT — `intents/profileIntents.js`

```js
export function saveProfileIntent(identityRef, payload, wrapperCtx) {
  return Object.freeze({
    kind: "profile.save",
    capability: "save",
    identityRef,
    payload,
    wrapperCtx,
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

### ACTION — `actions/profileActions.js`

```js
import { UserProfileForm } from "../forms/userProfile.js";

export function validateAndSave(state, intent) {
  const next = { ...state, ...intent.payload };
  const violations = UserProfileForm.invariants
    .map((inv) => inv(next))
    .filter((r) => r !== true);

  if (violations.length > 0) {
    return {
      delta: { _validationErrors: violations },
      effects: [],
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
  return {
    delta: { _resetToCommitted: true },
    effects: [],
  };
}
```

### EFFECT — (declared inline by Actions)

```js
// Effects are pure data returned by Actions. Example standalone value:
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
```

### IMPACT — `impacts/httpImpact.js`

```js
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
      if (effect.compensatingEffect) {
        await execute([effect.compensatingEffect]);
      }
      outcomes.push({ ok: false, error: err.message });
    }
  }
  return outcomes;
}
```

### WRAPPER — `wrappers/profileEditor.js`

```js
class ProfileEditor extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
  }

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

  get eventSlots() {
    return {
      submit: () => this.shadowRoot.querySelector("#profile-form"),
      cancel: () => this.shadowRoot.querySelector("#cancel"),
    };
  }

  readInputs() {
    const form = this.shadowRoot.querySelector("#profile-form");
    return Object.fromEntries(new FormData(form));
  }
}

customElements.define("profile-editor", ProfileEditor);
export { ProfileEditor };
```

### ADAPTER — `adapters/profileAdapter.js`

```js
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

### MANAGER — `manager/profileManager.js`

```js
import { editableFacet, readOnlyFacet } from "../facets/profileFacets.js";
import { applyDelta } from "../instances/profileInstance.js";
import { isExpired } from "../tokens/sessionToken.js";
import { execute as runImpact } from "../impacts/httpImpact.js";
import { bindProfileAdapter } from "../adapters/profileAdapter.js";

export function createProfileManager(wrapper, identity, token, instance) {
  let currentInstance = instance;
  let currentToken = token;

  function selectFacet() {
    if (!currentToken || isExpired(currentToken) || !currentToken.claims.canEdit) {
      return readOnlyFacet;
    }
    return editableFacet;
  }

  async function dispatch(intent) {
    const facet = selectFacet();
    const action = facet.delegates[intent.capability];

    if (!action) {
      console.warn(`No capability "${intent.capability}" in current facet`);
      return;
    }

    const { delta, effects } = action(currentInstance.state, intent);
    currentInstance = applyDelta(currentInstance, delta);
    wrapper.render(currentInstance);

    if (effects.length > 0) {
      const outcomes = await runImpact(effects);
      for (const outcome of outcomes) {
        if (outcome.followUpIntent) {
          await dispatch(outcome.followUpIntent);
        }
      }
    }
  }

  function bind() {
    wrapper.render(currentInstance);
    bindProfileAdapter(wrapper, identity.entityId, dispatch);
  }

  return { bind, dispatch };
}
```

### Entry Point — `app.js`

```js
import { createUserIdentity } from "./identity/userIdentity.js";
import { createProfileInstance } from "./instances/profileInstance.js";
import { issueSessionToken } from "./tokens/sessionToken.js";
import { createProfileManager } from "./manager/profileManager.js";

const identity = createUserIdentity("u-1234");
const token = issueSessionToken(identity, { canEdit: true, role: "owner" });
const instance = createProfileInstance(identity, {
  displayName: "Alice",
  email: "alice@example.com",
  bio: "Software engineer",
  avatarUrl: null,
});
const wrapper = document.querySelector("profile-editor");
const manager = createProfileManager(wrapper, identity, token, instance);
manager.bind();
```

**Key observations:**
- Every file maps to exactly one AXIS component. The folder structure *is* the architecture.
- Actions never touched `fetch` — they declared an Effect. Impact did the fetching.
- The Manager is ~40 lines: routing, committing, delegating. No business logic or IO.
- Token drives Facet selection — change `canEdit: false` and the save button still renders (Wrapper doesn't know), but `dispatch` finds no `save` capability in `readOnlyFacet`.
- Testing is trivial per layer: call `validateAndSave()` with test data → assert delta and effects. No mocking fetch, no rendering DOM.

---

## Worked Example: Extensible Web Server (Vanilla Node.js)

New routes and services can be dynamically added at runtime — without restarting, without modifying core files. Dynamic extensibility is just the Manager binding new Identities, Forms, and Facets into an already-running composition.

### FORM — `forms/serviceForm.js`

```js
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

export const RouteForm = {
  shape: {
    method:     { type: "string", enum: ["GET", "POST", "PUT", "DELETE", "PATCH"], required: true },
    path:       { type: "string", required: true },
    capability: { type: "string", required: true },
  },
  defaults: {
    method: "GET",
  },
  invariants: [],
};
```

### IDENTITY — `identity/serviceIdentity.js`

```js
export function createServiceIdentity(serviceName) {
  return Object.freeze({
    entityId: `service:${serviceName}`,
    formRef: "ServiceForm",
    versioningPolicy: "semver",
  });
}

export function createRouteIdentity(serviceIdentity, method, path) {
  return Object.freeze({
    entityId: `${serviceIdentity.entityId}/route:${method}:${path}`,
    formRef: "RouteForm",
    versioningPolicy: "hash",
  });
}
```

### INSTANCE — `instances/serverInstance.js`

```js
export function createServerInstance() {
  return {
    formRef: "ServerForm",
    identityRef: "server:main",
    revisionId: 0,
    state: {
      services: {},
      routeTable: [],
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

### TOKEN — `tokens/requestToken.js`

```js
export function issueRequestToken(req) {
  const authHeader = req.headers["authorization"] || "";
  const apiKey = req.headers["x-api-key"] || "";

  return {
    identityRef: extractCallerId(authHeader, apiKey),
    scope: "request",
    expiry: Date.now() + 30_000,
    claims: {
      roles: extractRoles(authHeader),
      rateLimit: { remaining: 100, window: 60_000 },
    },
  };
}

function extractCallerId(authHeader, apiKey) {
  if (apiKey) return `apikey:${apiKey.slice(0, 8)}`;
  if (authHeader) return `bearer:${hashToken(authHeader)}`;
  return "anonymous";
}

function extractRoles(authHeader) {
  if (!authHeader) return ["public"];
  return ["authenticated"];
}

function hashToken(s) { return s.slice(-8); }
```

### FACET — `facets/serviceFacets.js`

```js
import { registerService, unregisterService } from "../actions/managementActions.js";

export const managementFacet = {
  delegates: {
    "service.register":   registerService,
    "service.unregister": unregisterService,
  },
};

const serviceFacets = {};

export function registerServiceFacet(serviceId, facet) {
  serviceFacets[serviceId] = facet;
}

export function getServiceFacet(serviceId) {
  return serviceFacets[serviceId] || null;
}
```

### INTENT — `intents/serverIntents.js`

```js
export function requestIntent(method, path, body, token, wrapperCtx) {
  return Object.freeze({
    kind: "http.request",
    capability: null,
    identityRef: token.identityRef,
    payload: { method, path, body },
    wrapperCtx,
    meta: { timestamp: Date.now(), tokenScope: token.scope },
  });
}

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

### ACTION — `actions/managementActions.js`

```js
import { ServiceForm } from "../forms/serviceForm.js";

export function registerService(state, intent) {
  const config = intent.payload;
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

### EFFECT — (declared inline by Actions)

```js
// HTTP_RESPONSE — send a response to the caller
{ kind: "HTTP_RESPONSE", target: "caller", payload: { status: 200, body: { message: "Hello!" } } }

// LOG — emit an audit log entry
{ kind: "LOG", target: "audit", payload: { event: "service.registered", serviceId: "service:greeter" } }
```

### IMPACT — `impacts/serverImpact.js`

```js
const handlers = {
  HTTP_RESPONSE: async (effect, ctx) => {
    const { res } = ctx;
    res.writeHead(effect.payload.status, { "Content-Type": "application/json" });
    res.end(JSON.stringify(effect.payload.body));
    return { ok: true };
  },

  LOG: async (effect) => {
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

### WRAPPER — `wrappers/httpWrapper.js`

```js
import { createServer } from "node:http";

export function createHttpWrapper(port) {
  const server = createServer();
  let requestHandler = null;

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

### ADAPTER — `adapters/httpAdapter.js`

```js
import { requestIntent } from "../intents/serverIntents.js";
import { issueRequestToken } from "../tokens/requestToken.js";

export function bindHttpAdapter(wrapper, dispatch) {
  wrapper.onRequest(async (req, res) => {
    const body = await readBody(req);
    const token = issueRequestToken(req);
    const intent = requestIntent(req.method, req.url, body, token, "httpWrapper");
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

### MANAGER — `manager/serverManager.js`

```js
import { managementFacet, getServiceFacet, registerServiceFacet }
  from "../facets/serviceFacets.js";
import { createServerInstance, applyDelta } from "../instances/serverInstance.js";
import { execute as runImpact } from "../impacts/serverImpact.js";
import { bindHttpAdapter } from "../adapters/httpAdapter.js";

export function createServerManager(wrapper) {
  let instance = createServerInstance();

  function resolveRoute(method, path) {
    return instance.state.routeTable.find(
      (r) => r.method === method && matchPath(r.path, path)
    );
  }

  function matchPath(pattern, actual) {
    return pattern === actual;
  }

  async function dispatch(intent, ctx) {
    const { token } = ctx;
    let facet, capability;

    if (intent.kind.startsWith("management.")) {
      facet = managementFacet;
      capability = intent.capability;
    } else if (intent.kind === "http.request") {
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

    const { delta, effects } = action(instance.state, intent);

    if (Object.keys(delta).length > 0) {
      instance = applyDelta(instance, delta);
    }

    if (delta.registerService) {
      const svc = delta.registerService;
      registerServiceFacet(svc.identity.entityId, svc.facet);
    }

    await runImpact(effects, ctx);
  }

  function bind() {
    bindHttpAdapter(wrapper, dispatch);
  }

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

### Entry Point — `server.js`

```js
import { createHttpWrapper } from "./wrappers/httpWrapper.js";
import { createServerManager } from "./manager/serverManager.js";
import { createServiceIdentity } from "./identity/serviceIdentity.js";
import { greet, personalizedGreet } from "./actions/greeterActions.js";

const wrapper = createHttpWrapper(3000);
const manager = createServerManager(wrapper);
manager.bind();

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

await wrapper.start();

// GET  /hello       → "Hello, world!"
// POST /hello/name  → "Hello, Alice!"
//
// To add a new service at runtime, call manager.registerServiceDirect()
// from a plugin loader. No restart. No recompile. No core file changes.
```

**Key observations:**
- Dynamic extensibility is just state — registering a service = committing a delta to the server Instance.
- The Wrapper never routes — `http.createServer` hands `(req, res)` to the Adapter. All routing lives in the Manager.
- Each service is an Identity + Facet pair. The service doesn't need to know about the server, HTTP, or other services.
- Impact carries the `res` context — never seen by Actions (pure) or Facets (data). Only Impact's `HTTP_RESPONSE` handler touches it.
- Testing remains trivial: test `registerService()` with mock state → assert delta. Test `greet()` → assert HTTP_RESPONSE effect. No server, no sockets.

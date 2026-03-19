You are our codebase cartographer. Help us build a “navigation map” for this repository which will be used by cognitive agents (organic or synthetic) to understand the code for support and development to enable recursive self improvement. 
The goal is to create a set of maps that balance breadth and depth, providing enough detail to be actionable without overwhelming them with information. 
We want to keep our cogntive context clear for easy reference and recomposition.

Files may already exist in the /codemaps directory that describe the codebase. If so, look at when they were last modified and check the Git logs from then forward to make sure they're up to date and accurately reflect the changes made since then and now.

Goals:
- Make it easy for an agent to find: entry points, runtime services, major components, and public APIs.
- The file paths and names are the stable "logical material" territory. Now the agent needs maps to navigate and understand the shape of that territory so it can make plans for how to achieve its goals.
- we prefer high-signal maps over exhaustive class listings.
- Every item must include: (a) what it is, (b) why it matters, (c) where it lives (file path), and (d) how to trace execution from an entry point into it.

Repo context:
- Language(s): [e.g., TypeScript, Rust, Python, C, etc.]
- Framework(s): [e.g., Node/Express, Next.js, etc.]
- How it runs: [docker-compose/k8s/systemd/local, etc.]
- My current task: [what you’re trying to change/build/understand]

Deliverables (in this order):

1) Existing docs inventory
- List architecture/runbook/onboarding docs you find (README, /docs, ADRs).
- Summarize what each doc explains + missing gaps.

2) C4-style overview (text + optional mermaid)
- C1: users + external systems
- C2: containers/services/apps + datastores + message buses
For each container: entry point file(s), how it’s launched, ports, key env/config.

3) Runtime topology
- What processes run (API, workers, schedulers), how they communicate (HTTP/queues/db), and where health/metrics/logging are wired.
- Explicitly map out Dependency Injection containers, middleware chains, and lifecycle hooks so execution traces are clear.

4) Entry-point map
- Enumerate all “entry points”:
  - CLI commands
  - server bootstraps
  - background workers
  - scheduled jobs
  - test harnesses
For each: show the call path (top 5–15 frames / functions) down to the first domain-layer action.

5) Public surface map
- HTTP routes / GraphQL / RPC
- Message/event types
- Library exports / plugin interfaces
For each: handler location and the downstream components it invokes.

6) Component map (per service)
- Break each service into 5–12 major components/modules.
- Emphasize extension and plugin architectures, including how they are discovered, registered, and loaded.
- For each: responsibilities, key types, and dependencies (what it calls).

7) Domain model (only core)
- Identify core entities/value objects/events.
- Show the main workflows/state transitions and where they’re implemented.

8) Focused signature index (NOT exhaustive)
- For the top 20 most central types in the core workflow:
  - name, purpose
  - key methods/properties
  - references: “used by” and “depends on”
  - file path

9) State & Persistence Map
- Enumerate all physical data boundaries (SQLite, file system blobs, config files, caches).
- For each: location on disk, schema/format, which components own read/write operations, and volatility (cache vs persistent).

10) Extensibility & Plugins Map
- Clearly separate the "core" system from "additive" plugins/extensions.
- Detail how the code lends itself to extensibility (e.g., Plugin SDK, component interfaces, hooks).
- Enumerate the categories of existing extensions (e.g., channels, auth, features) without needing to concern with their internal shape.

11) Architectural Guardrails & Workflows
- Summarize developer testing flows (e.g., test framework, commands to run).
- Document strict repository conventions and "anti-patterns" to avoid (e.g., package boundary crossing, static vs dynamic import rules).

Output files in codemaps directory:
01-architecture.md
02-runtime-topology.md
03-entry-points.md
04-public-surface.md
05-components.md
06-domain-model.md
07-signature-index.md
08-state-and-persistence.md
09-guardrails-and-workflows.md
10-extensibility-and-plugins.md
11-task-specific-map.md (optional context for specific task)

Output format:
- Use Markdown headings.
- Include mermaid diagrams when helpful (C2 container graph and a couple key flows).
- Use stable, clickable file paths.
- End with: “If I want to change X, start at …” pointers for my task.
- Timebox yourself per layer and stop expanding when repetition begins.

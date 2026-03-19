You are my codebase cartographer. Help us build a “navigation map” for this repository which will be used by cognitive agents (organic or synthetic) to understand the code for support and development to enable recursive self improvement. The goal is to create a set of maps that balance breadth and depth, providing enough detail to be actionable without overwhelming with information so we can keep our cogntive context clear for easy reference and recomposition.

Files may already exist in the /codemaps directory that describe the codebase, if so, make sure they're up to date and accurate.

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

1) C4-style overview (text + optional mermaid)
- C1: users + external systems
- C2: containers/services/apps + datastores + message buses
For each container: entry point file(s), how it’s launched, ports, key env/config.

1) Runtime topology
- What processes run (API, workers, schedulers), how they communicate (HTTP/queues/db), and where health/metrics/logging are wired.

1) Entry-point map
- Enumerate all “entry points”:
  - CLI commands
  - server bootstraps
  - background workers
  - scheduled jobs
  - test harnesses
For each: show the call path (top 5–15 frames / functions) down to the first domain-layer action.

1) Public surface map
- HTTP routes / GraphQL / RPC
- message/event types
- library exports / plugin interfaces
For each: handler location and the downstream components it invokes.

1) Component map (per service)
- Break each service into 5–12 components/modules.
- For each: responsibilities, key types, and dependencies (what it calls).

1) Domain model (only core)
- Identify core entities/value objects/events.
- Show the main workflows/state transitions and where they’re implemented.

1) Focused signature index (NOT exhaustive)
- For the top 20 most central types in the core workflow:
  - name, purpose
  - key methods/properties
  - references: “used by” and “depends on”
  - file path

Output files in codemaps directory:
01-architecture.md
02-runtime-topology.md
03-entry-points.md
04-public-surface.md
05-components.md
06-domain-model.md
07-signature-index.md
08-task-specific-map.md

Output format:
- Use Markdown headings.
- Include mermaid diagrams when helpful (C2 container graph and a couple key flows).
- Use stable, clickable file paths.
- End with: “If I want to change X, start at …” pointers for my task.
- timebox yourself per layer and stop expanding when repetition begins
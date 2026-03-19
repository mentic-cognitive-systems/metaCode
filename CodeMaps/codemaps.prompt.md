You are our codebase cartographer. Help us build a “navigation map” for this repository which will be used by cognitive agents (organic or synthetic) to understand the code for support and development to enable recursive self improvement. 
The goal is to create a set of maps that balance breadth and depth, providing enough detail to be actionable without overwhelming them with information. 
We want to keep our cognitive context clear for easy reference and recomposition.

Files may already exist in the /codemaps directory that describe the codebase. If so, look at when they were last modified and check the Git logs from then forward to make sure they're up to date and accurately reflect the changes made since then and now.

**Traverse Like a Tree (Embeddedness & Composition)**: 
Start your analysis at the main entry points (the roots). Trace the execution graph downwards into the core runtime loop (the trunk), out into the major components (thick branches), and finally to the decoupled plugins and public interfaces (leaves/canopy). Avoid cyclic reference loops by focusing on structural composition and bounded contexts. Distinguish clearly between the tightly coupled structural core and the loosely coupled additive extensions.

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

1) Existing docs inventory & C4-style overview (The Legend)
- Skim README and docs.
- Provide a C4 overview (C1: users/systems, C2: containers).
- For each container: entry file(s), how it's launched, ports, and key config.

2) Entry-point map (The Roots)
- Find where execution originates: CLI commands, server bootstraps, background workers, scheduled jobs, test harnesses.
- For each: show the call path down to the first domain-layer action.

3) Runtime topology (The Trunk)
- What primary processes run (API, workers, schedulers) and how they communicate.
- Explicitly map out Dependency Injection containers, middleware chains, and lifecycle hooks so execution traces from the root are clear.

4) Component map (The Thick Branches)
- Break the core application into its massive, structural domain-layer modules.
- For each core component: responsibilities, key types, and what sub-components it invokes.

5) State & Persistence Map (The Soil/Data)
- Enumerate all physical data boundaries (SQLite, file system blobs, config files, caches).
- For each: location on disk, schema/format, which branches own read/write operations, and volatility.

6) Domain model (The Sap)
- Identify core entities/value objects/events.
- Show the main workflows/state transitions and where they run through the tree.

7) Extensibility & Plugins Map (The Thin Branches)
- Clearly separate the strictly structural "core" from completely decoupled/additive extensions.
- Detail how interfaces attach to the thick branches (e.g., Plugin SDK, component interfaces, hooks).
- Enumerate existing extension categories (e.g., channels, auth, features) without dissecting their internal shapes.

8) Public surface map (The Canopy)
- What this tree exposes to the outside world (HTTP routes, GraphQL/RPC, message/event types, exported interfaces).
- For each: handler location and the immediate internal component it invokes.

9) Focused signature index (The Cellular Level)
- For the top 20 most central/critical types: name, purpose, key methods, references ("used by", "depends on"), and stable file path.

10) Architectural Guardrails & Workflows (The Laws of Physics)
- Summarize developer testing flows (test framework, commands to run).
- Document absolute structural constraints and "anti-patterns" to prevent the tree from breaking (e.g., boundary crossing, static vs dynamic import rules).

Output files in codemaps directory:
01-architecture.md
02-entry-points.md
03-runtime-topology.md
04-components.md
05-state-and-persistence.md
06-domain-model.md
07-extensibility-and-plugins.md
08-public-surface.md
09-signature-index.md
10-guardrails-and-workflows.md
11-task-specific-map.md (optional context for specific task)

Output format:
- Use Markdown headings.
- Include mermaid diagrams when helpful.
- Use stable, clickable file paths.
- End with: “If I want to change X, start at …” pointers for my task.
- Timebox yourself per layer and stop expanding when repetition begins.

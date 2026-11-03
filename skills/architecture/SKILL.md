---
name: architecture
description: Analyze, map, and document application layers, communication rules, data flow, and component relationships in .agent/assets/architecture/.
disable-model-invocation: true
---

# Architecture

Analyze and document application layers, communication boundaries, and component responsibilities. Output stays high-level, language-agnostic, and points to key entry files rather than inlining code snippets.

## Style and Writing Guidelines

Write directly in the target tone from the start to save tokens and avoid secondary rewrite passes. Follow these rules from `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain, concrete words over abstract metaphors (Rules 7, 26, 31). Avoid "substrate", "tapestry", "leverage", "surface".
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Workflow

1. Discover entry points and folder structure across the codebase. Do not assume fixed directory names like `src/`.
2. Trace the data lifecycle: ingress/transport, core domain logic, persistence, and external egress.
3. Write or update `.agent/assets/architecture/system.md` using the template below.
4. For non-trivial components with internal state machines, public contracts, or multi-step pipelines, create `.agent/assets/architecture/modules/<module-name>.md`.
5. Keep explanations brief. Point to key file paths rather than pasting code snippets.

---

## Template for system.md

```markdown
# System architecture

## Overview
High-level statement of what this application does and its primary execution model (CLI, daemon, HTTP service, MCP server).

## Layers and responsibilities
Describe each layer in 2 to 3 sentences. Name the key files representing the boundary.

- **Ingress / Transport**: Handles incoming protocols, CLI flags, network connections, or RPC calls. Key files: `<paths>`
- **Core / Domain logic**: Encapsulates business rules, orchestration, and state transitions. Free of transport-specific concerns. Key files: `<paths>`
- **Egress / Integrations**: External clients, database persistence, third-party APIs, and hardware/OS interfaces. Key files: `<paths>`

## Communication rules
Define strict boundaries between layers.
- Allowed directions of dependency (e.g., Ingress calls Core; Core calls Egress interfaces; Egress never depends on Ingress).
- Forbidden interactions (e.g., circular dependencies, raw transport objects leaking into domain logic).

## Data flow and lifecycle
- **Ingress validation**: Where raw external payloads enter, validate, and convert into internal domain types.
- **State and persistence**: How and where runtime state lives (in-memory, embedded store, relational DB).
- **Error propagation**: How failures surface across boundaries.

## Component map
Summary table of major functional areas:

| Component | Responsibility | Entry file / directory |
| :--- | :--- | :--- |
| `<name>` | `<1 sentence summary>` | `<path>` |
```

---

## Template for modules/<module-name>.md

```markdown
# Component: <module-name>

## Purpose
2 to 3 sentences on what this component owns and why it exists as an independent boundary.

## Boundaries
- **Imports from**: What other internal components or external packages this module depends on.
- **Exposes**: Key public types, interfaces, or entry functions.
- **Hidden internal state**: Details and invariants the rest of the application must not touch directly.

## Data handling
- Primary inputs and outputs.
- Validation and boundary checks enforced here.

## Key files
- `<path>`: <what it does>
```

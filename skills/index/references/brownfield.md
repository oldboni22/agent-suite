# Brownfield initialization procedure

Initialize framework assets for an established repository with existing source code and package manifests.

## Guiding principle

An established codebase contains concrete technical artifacts, but business intent and invariants remain implicit. The agent surveys technical files first to avoid asking for discoverable facts. The user guides exploration through critical subsystems and defines business domain reality.

## Procedure

1. **Technical survey and entry points**:
   - Inspect top-level files and directories.
   - Read package manifests (`go.mod`, `package.json`, `Cargo.toml`, `pyproject.toml`) to determine runtime versions, dependencies, and project root.
   - Locate primary entry points (`main.go`, `index.ts`, `app.py`, `src/`) and configuration files.
   - Identify existing build, lint, and test scripts.

2. **User-guided structural exploration**:
   - Ask the user to identify the primary functional areas and highlight critical paths.
   - Explore directories incrementally based on user guidance rather than reading all files into context at once.
   - Identify ingress adapters, core business logic, and egress integrations.
   - Write `.agent/assets/architecture/system.md` using the template from `architecture`.
   - For major self-contained subsystems, write `.agent/assets/architecture/modules/<module-name>.md`.

3. **Domain elicitation and grill gate**:
   - Extract candidate domain entities, states, and external API integrations from public types and interfaces.
   - Run the `grill-me` workflow sequentially with the user, strictly observing `honest` rules (no flattery, no performative apologies, objective pushback).
   - Require the user to define the core business problem, primary actors, and universal business invariants.
   - Require the user to confirm entity lifecycles and validation rules for critical workflows.
   - Write `.agent/assets/domain/general.md` and relevant topic files under `.agent/assets/domain/topics/`.

4. **Testing conventions capture**:
   - Inspect existing test files, test configurations, and CI scripts.
   - Extract primary test execution commands, verbose test commands, and coverage flags.
   - Check if dedicated tool suites exist (race detection, test linters, mutation tools).
   - Document table-driven test patterns, fixture locations, and mocking conventions.
   - Write `.agent/assets/testing/conventions.md`.

5. **Language conventions capture**:
   - Extract project-specific code style, formatting tools, and error-handling patterns.
   - Document common anti-patterns or LLM blind spots observed in the codebase.
   - Write `.agent/assets/language/<lang>.md`.

6. **Asset index manifest generation**:
   - Generate `.agent/assets/index.md`.
   - List canonical CLI commands (build, lint, test) and relative asset pointers.
   - Do not create or overwrite user-owned root files (`AGENT.md`, `CLAUDE.md`).

7. **Style audit**:
   - Validate every generated asset file against `unslop` and `honest` rules.

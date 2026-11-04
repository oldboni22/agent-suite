# Greenfield initialization procedure

Initialize framework assets for an empty repository or a brand new project with zero existing source code.

## Guiding principle

In a greenfield project, no code, tests, or build manifests exist to reverse-engineer. Business domain and system intent exist entirely in the user's mind and project specifications. The agent must interview the user methodically rather than inventing requirements from thin air.

## Procedure

1. **Domain interview**:
   - Run the `grill-me` workflow to establish the business problem, observing `honest` rules.
   - Ask the user to define the core purpose of the application and the problem it solves.
   - Elicit primary real-world actors, external system integrations, and universal business invariants.
   - Write `.agent/assets/domain/general.md` using the template from `.agent/skills/domain/SKILL.md`.

2. **System topology interview**:
   - Ask the user to specify the application type (CLI tool, background daemon, HTTP service, MCP server).
   - Elicit the intended high-level layers: ingress/transport, core domain logic, and external egress.
   - Elicit forbidden dependency directions and communication rules.
   - Write `.agent/assets/architecture/system.md` using the template from `.agent/skills/architecture/SKILL.md`.
   - Defer granular module architecture files (`modules/*.md`) until modules are implemented.

3. **Language and runtime stack interview**:
   - Ask the user to specify the target programming language, package manager, and language version.
   - Elicit idiomatic conventions, preferred standard library defaults, and project-specific guardrails.
   - Write `.agent/assets/language/<lang>.md` using the template from `.agent/skills/language/SKILL.md`.

4. **Asset index manifest generation**:
   - Confirm expected build, compile, and run commands with the user.
   - Generate `.agent/assets/index.md` with `last_synced_commit: <sha>` on line 1.
   - List the canonical build, lint, and run commands and direct links to active living assets.
   - Defer testing assets until testing tools and runners are introduced during feature implementation.
   - Do not create or overwrite user-owned root files (`AGENT.md`, `CLAUDE.md`).

5. **Style audit**:
   - Verify every newly written asset complies with `.agent/skills/unslop/SKILL.md` and `.agent/skills/honest/SKILL.md`.
   - Remove any temporary scratch files outside `.agent/assets/`.

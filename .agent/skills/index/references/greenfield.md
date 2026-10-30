# Greenfield initialization procedure

Initialize framework assets for an empty repository or a brand new project with zero existing source code.

## Guiding principle

In a greenfield project, no code, tests, or build manifests exist to reverse-engineer. Business domain and system intent exist entirely in the user's mind and project specifications. The agent must interview the user methodically rather than inventing requirements from thin air.

## Procedure

1. **Domain interview**:
   - Run the `grill-me` workflow to establish the business problem.
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

4. **Operational commands and root runbook**:
   - Confirm expected build, compile, and run commands with the user.
   - Create `AGENT.md` at the repository root.
   - Follow the root runbook structure: framework pointer to `.agent/README.md`, working directory rules, unslop compliance, zero guessing, and canonical CLI commands.
   - Defer testing assets until testing tools and runners are introduced during feature implementation.

5. **Style audit**:
   - Verify every newly written asset and root `AGENT.md` complies with `.agent/skills/unslop/SKILL.md`.
   - Remove any temporary scratch files outside `.agent/assets/` and root `AGENT.md`.

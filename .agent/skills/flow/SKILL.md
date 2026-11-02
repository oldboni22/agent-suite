---
name: flow
description: Configure and manage repository lifecycle pipelines, stage opt-outs, and task execution sequences in .agent/assets/flow/lifecycle.md and .agent/scope/intro.md.
disable-model-invocation: true
---

# Flow

Configure and manage the execution lifecycle of tasks across the repository. The flow skill establishes the permanent lifecycle contract in `.agent/assets/flow/lifecycle.md`, provides declarative opt-out mechanisms for pipeline stages (such as skipping automated tests or verifications), and translates repository flow policies into active task pipeline checklists in `.agent/scope/intro.md`.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Core invariants

- The repository lifecycle contract lives in `.agent/assets/flow/lifecycle.md`.
- Opting out of any stage (such as testing or verification) means removing or commenting out that stage in `lifecycle.md` for permanent repository policy, or omitting it from `intro.md` for a single task.
- Active task execution state is tracked via a pipeline checklist inside `.agent/scope/intro.md`. Downstream agents never invent extra state tracking files.
- The host orchestrator executes pipeline stages in the exact order declared in `intro.md`.

## Lifecycle configuration workflow

1. **Inspect repository state**:
   - Check if `.agent/assets/flow/lifecycle.md` exists.
   - Inspect `.agent/assets/testing/conventions.md` to see if a test harness is configured.
   - If `conventions.md` declares no test harness or testing is explicitly disabled, default the pipeline to the lean flow (omitting `tester` and `test-verifier`).

2. **Establish or update lifecycle asset**:
   - Write or update `.agent/assets/flow/lifecycle.md` using the template below.
   - Document default pipelines and available task profiles.

3. **Task-level pipeline synthesis (for planner)**:
   - Read `.agent/assets/flow/lifecycle.md`.
   - Resolve the active stages for the current task based on user intent and task scope.
   - Write the resolved stages as a checklist into the `## Pipeline` section of `.agent/scope/intro.md`.

---

## Template for lifecycle.md

```markdown
# Repository lifecycle flow

## Default pipeline
1. `planner`: Discovers context, resolves ambiguities via grill-me, and emits scope contracts.
2. `coder`: Executes atomic implementation steps surgically on the task branch.
3. `code-verifier`: Dispatches isolated auditor to verify code diffs, scope boundaries, and invariants.
4. `tester`: Dispatches isolated worker to author tests conforming to testing conventions.
5. `test-verifier`: Dispatches isolated auditor to run mutation checks and verify regression resistance.
6. `closure`: Synchronizes living assets, creates conventional commit, and cleans up worktrees.

## Available profiles

### Standard feature profile
Use for standard features, bug fixes, and refactors that require test authoring.
- Stages: `planner`, `coder`, `code-verifier`, `tester`, `test-verifier`, `closure`

### Lean profile (no automated tests)
Use when the repository has no test harness, for exploratory prototypes, or when testing is explicitly deferred.
- Stages: `planner`, `coder`, `code-verifier`, `closure`

### Test-only profile
Use when retrofitting tests or increasing test coverage without changing production code.
- Stages: `planner`, `tester`, `test-verifier`, `closure`

### Documentation and asset profile
Use for repository knowledge base updates, architecture notes, and documentation changes.
- Stages: `planner`, `closure`

## Opt-out rules
- To disable a stage permanently for the repository, remove the stage from the default pipeline above.
- To disable a stage for a specific task, omit the stage from the `## Pipeline` checklist in `.agent/scope/intro.md` during planning.
```

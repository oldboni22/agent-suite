---
name: planner
description: Plan tasks and features into .agent/scope/ (intro.md, coder-plan.md, closure.md) using codebase discovery, asset inspection, flow configuration, and relentless grilling to eliminate ambiguity.
disable-model-invocation: true
---

# Planner

Plan features, refactors, and fixes by producing structured execution artifacts in `.agent/scope/`. The planner reads existing architecture and decision records, checks for unindexed human commits, explores the codebase to find boundaries and redlines, grills the user until all dependencies are settled, configures the execution pipeline from `.agent/assets/flow/lifecycle.md`, and writes explicit step-by-step instructions for downstream agents.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Core invariants

- Implementation plans must not exceed 5 atomic steps per `coder-plan.md`. Tasks exceeding 5 steps must be decomposed into sequential sub-tasks executed via separate commits on the same branch.
- Execution mode must be explicitly declared in `intro.md`: `in-place` (default for interactive single-task developer sessions) or `worktree` (for parallel tasks, background tasks, or dirty workspaces).
- Every `intro.md` must contain an active `## Pipeline` checklist derived from `.agent/assets/flow/lifecycle.md`.
- Planners must check `AGENT.md` for `last_synced_commit`. If human commits occurred after that SHA, incorporate observed changes or prompt the user to resync assets.

## Workflow

1. **Asset drift check and context discovery**:
   - Inspect root `AGENT.md` for `last_synced_commit: <sha>`. If present, run `git log <sha>..HEAD --oneline` to detect human commits that modified source code without updating assets.
   - Inspect `.agent/assets/architecture/system.md` and module files to determine component boundaries.
   - Inspect `.agent/assets/why/` to identify non-negotiable invariants and past technical decisions.
   - Inspect `.agent/assets/language/<lang>.md` to identify project-specific language idioms and standards.
   - Inspect `.agent/assets/domain/general.md` and topic files under `.agent/assets/domain/topics/` to identify business invariants.
   - Inspect `.agent/assets/flow/lifecycle.md` to identify configured execution pipelines and available profiles.
   - Search the codebase to map call chains, imports, and touchpoint files.

2. **Execution mode selection**:
   - Check workspace status with `git status --porcelain`.
   - If this is a single interactive session and the workspace is clean: select `in-place` mode (`git checkout -b task/<task-id>`).
   - If running concurrent parallel tasks, background executions, or the primary working tree has uncommitted user edits: select `worktree` mode (`git worktree add .worktrees/<task-id> -b task/<task-id>`).

3. **Grill gate**:
   - Run the `grill-me` skill workflow following the format and rules in `.agent/skills/grill-me/SKILL.md`.
   - Engage in architectural sparring mode during exploratory discussion.
   - Transition to structured frontier mode once ideas converge. Ask decisions sequentially, presenting one question per turn.
   - Confirm pipeline profile selection with the user (e.g. standard vs. lean no-test profile).

4. **Plan decomposition ceiling**:
   - Enforce a strict ceiling of maximum 5 atomic steps per `coder-plan.md`.
   - If the task requires more than 5 steps, split the work into sequential sub-tasks (e.g. Sub-task 1: domain models, Sub-task 2: service layer).
   - Document the sub-task sequence in `intro.md`. Each sub-task will execute its own 5-step cycle and produce an intermediate commit on the task branch.

5. **Emit scope artifacts**:
   - Initialize `.agent/scope/`.
   - Write `.agent/scope/intro.md` using the template below.
   - Write `.agent/scope/coder-plan.md` using the template below (for sub-task 1 or the single task).
   - Write `.agent/scope/closure.md` using the template below.

---

## Template for intro.md

```markdown
# Task: <short title>

## Execution mode
- Mode: <in-place | worktree>
- Branch: task/<task-id>
- Path: <repository root, or .worktrees/<task-id>>

## Pipeline
- [ ] planner
- [ ] coder
- [ ] code-verifier
- [ ] tester
- [ ] test-verifier
- [ ] closure

## Problem
1 to 2 sentences stating the problem, bug, or feature request.

## Scope boundary
Explicit statement of what is included and what is excluded.
- **In scope**: <concrete deliverables and behaviors>
- **Out of scope**: <forbidden additions, non-goals, and deferred tasks>

## High-level approach
2 to 3 sentences outlining the technical path and affected subsystems.

## Sub-task decomposition
<If task exceeds 5 steps, list sequential sub-tasks. Each sub-task receives a dedicated commit on the task branch. If <= 5 steps, state "Single execution cycle".>
1. Sub-task 1: <summary> (commits upon code verification)
2. Sub-task 2: <summary> (commits upon code verification)

## Invariants and constraints
Non-negotiable rules derived from .agent/assets/why/ and project architecture that must not be broken.
- <invariant 1>
- <invariant 2>
```

---

## Template for coder-plan.md

```markdown
# Coder plan: <short title>

## Constraints and invariants
- **Invariants**: <non-negotiable rules derived from .agent/assets/why/ and project architecture>
- **Out of scope**: <explicit forbidden additions, non-goals, and deferred changes>
- **Testing policy**: Do not create new unit tests in src/. Features must not introduce new test files.
- **Step ceiling**: Maximum 5 atomic steps per plan.

## Pre-flight context
- [ ] Read and review .agent/assets/language/<lang>.md for language idioms and conventions.
- [ ] Review relevant architecture boundaries in .agent/assets/architecture/.
- [ ] Confirm no new unit tests will be created during this feature task.

## Execution steps

### Step 1: <step title>
- **Files to modify**: `<path>`
- **Action**: <concrete change to make>
- **Language rules to follow**: <specific rule or pattern from language assets>
- **Verification command**: `<build, compile, or lint command, or existing test command>`

### Step 2: <step title>
- **Files to modify**: `<path>`
- **Action**: <concrete change to make>
- **Language rules to follow**: <specific rule or pattern from language assets>
- **Verification command**: `<build, compile, or lint command, or existing test command>`

## Verification rules
- Run the verification command after each step.
- Do not proceed to the next step if verification fails.
- Do not create new unit tests in src/. If existing tests fail due to intentional contract changes, update them to match.
```

---

## Template for closure.md

```markdown
# Task closure: <short title>

## Documentation and assets to update
List what must be created or updated in .agent/assets/ when implementation succeeds.
- **Architecture**: <update system.md or create modules/<name>.md, or "None">
- **Decisions (why)**: <create .agent/assets/why/<component>/<decision>.md, or "None">
- **Language guide**: <add newly introduced project idiom to .agent/assets/language/<lang>.md, or "None">
- **Domain**: <update general.md or create topics/<topic>.md, or "None">
- **Flow**: <update lifecycle.md if new stage or profile was introduced, or "None">

## Handoff checklist
- [ ] All atomic steps in coder-plan.md verified and passed.
- [ ] Diff inspected against scope boundary in intro.md.
- [ ] Required asset updates in .agent/assets/ completed.
- [ ] Root AGENT.md last_synced_commit updated.
- [ ] Execution mode cleaned up (worktree pruned if in worktree mode).
```

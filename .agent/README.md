# Agent framework

Universal portable framework for autonomous agent planning, implementation, verification, and knowledge management.

## Philosophy

- Zero guessing: grill the user until all architectural boundaries, constraints, and dependencies are settled.
- Strict role separation: planners plan, coders code, and verifiers audit. Agents never self-audit their own diffs.
- Modular lifecycle flow: the repository flow contract in `.agent/assets/flow/lifecycle.md` governs pipeline stages. Teams can opt out of stages (such as automated tests) declaratively.
- Dual execution modes: single interactive tasks run in-place on task branches for full IDE indexing and editor ergonomics. Concurrent or background tasks run in isolated git worktrees (`.worktrees/<task-id>`).
- Decomposed execution: implementation plans enforce a ceiling of 5 atomic steps per cycle. Larger tasks decompose into sequential milestones committed on the same task branch.
- Dedicated test engineering: when testing is enabled, feature coders never write test files in `src/`. Dedicated testers author tests, and test verifiers audit regression power through native mutation runners or atomic synthetic perturbations.
- Ephemeral task scope: task execution state lives in `.agent/scope/` and archives into the task branch commit at completion.
- Persistent living assets: architectural boundaries, decisions (why), language idioms, testing conventions, and lifecycle flows live in `.agent/assets/`.
- Continuous drift detection: root `AGENT.md` tracks `last_synced_commit` to detect unindexed human commits.
- Style source of truth: all output adheres to the concise, plain-language rules in `.agent/skills/unslop/SKILL.md`.

## Directory layout

```text
.agent/
  README.md            # Portable framework specification (this document)
  skills/              # Reusable agent procedures and roles
    architecture/      # Maps boundaries and layers in .agent/assets/architecture/
    closure/           # Finalizes verified tasks, syncs assets, and cleans worktrees
    code-verifier/     # Dispatches isolated worker to audit production code diffs
    coder/             # Dispatches isolated worker to execute coder-plan.md
    domain/            # Discovers and structures domain entities and business rules
    flow/              # Configures lifecycle pipelines and declarative stage opt-outs
    grill-me/          # Relentless design tree interrogation and architectural sparring
    index/             # Initializes and synchronizes living assets and root AGENT.md
    language/          # Curates language idioms in .agent/assets/language/
    note/              # Captures mid-flight user hints into .agent/scope/notes.md
    planner/           # Discovers context, grills, and creates scope contracts
    test-verifier/     # Audits test suites and runs atomic mutation checks
    tester/            # Plans scenarios and dispatches isolated worker to write tests
    unslop/            # Writing style and plain-language rules
    why/               # Records architectural decisions in .agent/assets/why/
  assets/              # Long-lived repository knowledge base
    architecture/      # System architecture and module boundaries
    domain/            # Business terminology, workflows, and invariants
    flow/              # Repository lifecycle flow and pipeline profiles
    language/          # Language-specific conventions and LLM blind spots
    testing/           # Testing conventions, mocking standards, and tool suites
    why/               # Architectural decisions, trade-offs, and invariants
  scope/               # Transient active task workspace (git-ignored)
    intro.md           # Task definition, execution mode, pipeline checklist, and invariants
    coder-plan.md      # Atomic step-by-step implementation plan (max 5 steps)
    test-plan.md       # Scenarios, table cases, edge cases, and mocking boundaries
    closure.md         # Documentation sync instructions and cleanup checklist
    notes.md           # Mid-flight user hints and constraints
    scratch/           # Optional throwaway scripts for coder runtime verification
    run/               # Execution audit trail
      progress.md      # Feature step execution log and iteration counter (max 3 attempts)
      verdict.md       # Production code audit verdict (max 3 rounds)
      test-progress.md # Test step execution log and iteration counter (max 3 attempts)
      test-verdict.md  # Test audit verdict and atomic mutation log (max 3 rounds)
.worktrees/            # Isolated git worktrees for concurrent tasks (git-ignored)
```

## Task lifecycle state machine

A task moves through deterministic phases governed by the `## Pipeline` checklist in `.agent/scope/intro.md`:

1. **Discovery and planning (`planner`)**:
   - Inspect root `AGENT.md` for `last_synced_commit` to detect unindexed human code drift.
   - Inspect `.agent/assets/` (`flow/`, `architecture/`, `domain/`, `why/`, `language/`, `testing/`).
   - Select execution mode: in-place branch (`git checkout -b task/<task-id>`) for single interactive sessions, or isolated worktree (`git worktree add .worktrees/<task-id> -b task/<task-id>`) for concurrent tasks.
   - Run `grill-me` (sparring mode for discussion, structured frontier mode for decisions).
   - If plan exceeds 5 steps, decompose into sequential sub-tasks with intermediate commits.
   - Emit `.agent/scope/intro.md` (with active pipeline checklist), `.agent/scope/coder-plan.md`, and `.agent/scope/closure.md`.

2. **Feature implementation (`coder`)**:
   - Dispatch an isolated worker with local environment access.
   - Ingest `coder-plan.md` and relevant `language/<lang>.md`. Confirm step count is 5 or fewer.
   - Execute atomic steps in sequence. Perform surgical, in-place edits.
   - Run verification command after each step.
   - Log iteration attempts in `.agent/scope/run/progress.md`. If any step fails 3 times, halt immediately with `BLOCKED: STEP_ITERATION_LIMIT_EXCEEDED`.
   - Never create new unit test files in `src/`. If runtime experimentation is required, keep throwaway scripts strictly in `.agent/scope/scratch/`.
   - If executing a decomposed sub-task milestone, stage and commit the milestone on the task branch upon passing verification.
   - Exit with status `COMPLETED` or `BLOCKED`. Never self-audit diffs.

3. **Feature audit and verification (`code-verifier`)**:
   - Dispatch a fresh, isolated auditor worker with local environment access.
   - Inspect git deltas via `git status` and `git diff`.
   - Evaluate against 5 check categories: Scope boundaries, Invariants, Language conventions, Testing policy (confirm zero test files added to `src/`), and Build/existing tests.
   - Record evaluation round in `.agent/scope/run/verdict.md`.
   - If round count < 3 and verdict is `FAIL`, document actionable findings and return to implementation.
   - If round count >= 3 and any check fails, mark overall verdict as `BLOCKED` and escalate to user and planner.
   - If verdict is `PASS`, advance to the next pipeline stage declared in `intro.md`.

4. **Test planning and authoring (`tester`)** *(Optional per lifecycle profile)*:
   - If omitted from the task pipeline in `intro.md`, bypass directly to closure.
   - Inspect verified feature diffs or task specification in `intro.md`.
   - Read `.agent/assets/testing/conventions.md`.
   - Write `.agent/scope/test-plan.md` detailing scenarios, table-driven cases, edge cases, and mocks.
   - Dispatch an isolated worker subagent with local environment access.
   - Execute atomic test steps in sequence. Edit test files and fixtures only.
   - If a production bug is detected, halt immediately with status `BLOCKED: PRODUCTION_BUG_DETECTED` and escalate.
   - Log step execution in `.agent/scope/run/test-progress.md` (maximum 3 attempts per step).
   - Exit with status `COMPLETED` or `BLOCKED`.

5. **Test audit and mutation verification (`test-verifier`)** *(Optional per lifecycle profile)*:
   - Dispatch an isolated auditor worker with local environment access.
   - Check file boundaries: confirm zero modifications to production source code in `src/`.
   - Run repository tool suites specified in `.agent/assets/testing/conventions.md`.
   - Execute atomic mutation checks: run native mutation runners if configured; otherwise apply a temporary perturbation to production logic, confirm test fails RED, revert immediately (`git checkout -- <file>`), and confirm return to GREEN.
   - Record evaluation round in `.agent/scope/run/test-verdict.md`.
   - If a genuine production defect is confirmed, mark verdict as `BLOCKED: PRODUCTION_BUG_DETECTED` and halt immediately.
   - If round count < 3 and test audit fails, return actionable findings to `tester`.
   - If round count >= 3 and checks fail, mark `BLOCKED` and escalate.
   - If verdict is `PASS`, advance to closure.

6. **Closure and handoff (`closure`)**:
   - Confirm all required pipeline verdicts in `intro.md` are `PASS`.
   - Verify and synchronize living assets declared in `.agent/scope/closure.md`.
   - Validate modified asset files against `.agent/skills/unslop/SKILL.md`.
   - Force-stage `.agent/scope/` and stage all changes (`git add -f .agent/scope/ && git add .`).
   - Create an unslop-compliant conventional commit on `task/<task-id>`.
   - Update `last_synced_commit: <sha>` in root `AGENT.md`.
   - Clean up execution mode: prune worktree if in worktree mode (`git worktree remove` and `git worktree prune`).
   - Report the commit hash and branch handoff instructions to the user.

## Resume protocol

When starting or reviving a session, inspect the filesystem to determine active state:

| Condition | Inferred phase | Action |
| :--- | :--- | :--- |
| `AGENT.md` is missing or `.agent/assets/` is empty | `UNINITIALIZED` | Run `index` to bootstrap assets, flow lifecycle, and root runbook. |
| `.agent/scope/` is empty or missing | `IDLE` | Await user request, then run `planner`. |
| Dirty production files remain from crashed mutation run | `MUTATION_CLEANUP` | Run `git checkout -- .` to restore clean production state before resuming. |
| `coder-plan.md` exists, `run/progress.md` is missing or `IN_PROGRESS` | `IMPLEMENTATION` | Spawn `coder` to resume at the first incomplete step. |
| `run/progress.md` is `BLOCKED` or `BLOCKED: STEP_ITERATION_LIMIT_EXCEEDED` | `BLOCKED` | Present blocker hypothesis from `progress.md` to user. Discuss with `planner`. |
| `run/progress.md` is `COMPLETED`, `run/verdict.md` missing or `FAIL` | `VERIFICATION` / `FIX` | If verdict is missing, run `code-verifier`. If verdict is `FAIL`, spawn `coder` on actionable findings. |
| `run/verdict.md` is `PASS`, pipeline includes `tester`, `run/test-progress.md` missing or `IN_PROGRESS` | `TEST_AUTHORING` | Spawn `tester` worker to resume at first incomplete test step. |
| `run/verdict.md` is `PASS`, pipeline omits `tester` | `CLOSURE` | Run `closure` directly. |
| `run/test-progress.md` is `BLOCKED: PRODUCTION_BUG_DETECTED` | `PRODUCTION_BUG` | Present bug reproduction details to user. Discuss with `planner`. |
| `run/test-progress.md` is `BLOCKED` | `BLOCKED` | Present test authoring blocker hypothesis to user. |
| `run/test-progress.md` is `COMPLETED`, `run/test-verdict.md` missing or `FAIL` | `TEST_AUDIT` / `TEST_FIX` | If test-verdict is missing, run `test-verifier`. If `FAIL`, spawn `tester` on actionable findings. |
| `run/test-verdict.md` is `BLOCKED: PRODUCTION_BUG_DETECTED` | `PRODUCTION_BUG` | Present defect audit findings to user. Discuss with `planner`. |
| `run/test-verdict.md` is `PASS` | `CLOSURE` | Run `closure` to sync assets, commit, and prune worktrees. |
| `run/verdict.md` or `run/test-verdict.md` is `BLOCKED` | `BLOCKED` | Present audit findings from verdict files to user. Discuss with `planner`. |

## Core invariants

- Features must not add new unit test files to `src/`. Testing is handled exclusively by dedicated testing agents when configured in the pipeline.
- If runtime experimentation is needed during feature coding, throwaway scripts must live strictly inside `.agent/scope/scratch/` and never enter `src/`.
- Testers must never modify production source code in `src/`. If production code fails a valid test, escalate immediately.
- Agents must never audit their own diffs. Audits require isolated verifiers (`code-verifier` and `test-verifier`).
- Newly created tests must prove regression detection through native mutation tool suites or atomic synthetic perturbations.
- File modifications must be surgical. Avoid full-file overwrites unless creating new files.
- The repository git status must remain clean of untracked scratch files outside `.agent/scope/`.
- Tasks executing in worktrees must never merge into main automatically. Merges remain user-controlled.
- Code comments must be functional and unslop-compliant. Never write noisy comments that merely narrate or restate code.
- Every implementation plan is capped at 5 atomic steps. Multi-step features decompose into sequential milestones committed on the same task branch.
- Any step hitting 3 failed iterations halts immediately with `BLOCKED: STEP_ITERATION_LIMIT_EXCEEDED` to prevent rabbit-holing.

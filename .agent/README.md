# Agent framework

Universal portable framework for autonomous agent planning, implementation, verification, and knowledge management.

## Philosophy

- Zero guessing: grill the user until all architectural boundaries, constraints, and dependencies are settled.
- Strict role separation: planners plan, coders code, and verifiers audit. Agents never self-audit their own diffs.
- Dedicated test engineering: feature coders never write test files. Dedicated testers plan and author tests, and test verifiers audit regression power through synthetic mutation checks.
- Isolated execution: concurrent or background tasks run in dedicated git worktrees (`.worktrees/<task-id>`).
- Ephemeral task scope: task execution state lives in `.agent/scope/` and archives into the task branch commit at completion.
- Persistent living assets: architectural boundaries, architectural decisions (why), language idioms, and testing conventions live in `.agent/assets/`.
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
    grill-me/          # Relentless design tree interrogation
    index/             # Initializes and synchronizes living assets and root AGENT.md
    language/          # Curates language idioms in .agent/assets/language/
    note/              # Captures mid-flight user hints into .agent/scope/notes.md
    planner/           # Discovers context, grills, and creates scope contracts
    test-verifier/     # Audits test suites and runs synthetic mutation red-green checks
    tester/            # Plans scenarios and dispatches isolated worker to write tests
    unslop/            # Writing style and plain-language rules
    why/               # Records architectural decisions in .agent/assets/why/
  assets/              # Long-lived repository knowledge base
    architecture/      # System architecture and module boundaries
    domain/            # Business terminology, workflows, and invariants
    language/          # Language-specific conventions and LLM blind spots
    testing/           # Testing conventions, mocking standards, and tool suites
    why/               # Architectural decisions, trade-offs, and invariants
  scope/               # Transient active task workspace (git-ignored)
    intro.md           # Task definition, scope boundary, and invariants
    coder-plan.md      # Atomic step-by-step implementation plan
    test-plan.md       # Scenarios, table cases, edge cases, and mocking boundaries
    closure.md         # Documentation sync instructions and cleanup checklist
    notes.md           # Mid-flight user hints and constraints
    run/               # Execution audit trail
      progress.md      # Feature step execution log and iteration counter (max 3 attempts)
      verdict.md       # Production code audit verdict (max 3 rounds)
      test-progress.md # Test step execution log and iteration counter (max 3 attempts)
      test-verdict.md  # Test audit verdict and synthetic mutation log (max 3 rounds)
.worktrees/            # Isolated git worktrees for tasks (git-ignored)
```

## Task lifecycle state machine

A task moves through deterministic phases:

1. **Discovery and planning (`planner`)**:
   - For isolated execution, create a task worktree: `git worktree add .worktrees/<task-id> -b task/<task-id>`.
   - Inspect `.agent/assets/` (`architecture/`, `domain/`, `why/`, `language/`, `testing/`).
   - Run `grill-me` to resolve design choices, boundaries, and trade-offs sequentially.
   - Emit `.agent/scope/intro.md`, `.agent/scope/coder-plan.md`, and `.agent/scope/closure.md`.

2. **Feature implementation (`coder`)**:
   - Dispatch an isolated worker with local environment access (via platform subagents or CLI context fences).
   - Ingest `coder-plan.md` and relevant `language/<lang>.md`.
   - Execute atomic steps in sequence. Perform surgical, in-place edits.
   - Run verification command after each step.
   - Log iteration attempts in `.agent/scope/run/progress.md` (maximum 3 attempts per step).
   - Never create new unit test files or test functions during feature work.
   - Exit with status `COMPLETED` or `BLOCKED`. Never self-audit diffs.

3. **Feature audit and verification (`code-verifier`)**:
   - Dispatch a fresh, isolated auditor worker with local environment access.
   - Inspect git deltas via `git status` and `git diff`.
   - Evaluate against 5 check categories: Scope boundaries, Invariants, Language conventions, Testing policy, and Build/existing tests.
   - Record evaluation round in `.agent/scope/run/verdict.md`.
   - If round count < 3 and verdict is `FAIL`, document actionable findings and return to implementation.
   - If round count >= 3 and any check fails, mark overall verdict as `BLOCKED` and escalate to user and planner.
   - If verdict is `PASS`, proceed to test engineering or closure.

4. **Test planning and authoring (`tester`)**:
   - Inspect verified feature diffs or task specification in `intro.md`.
   - Read or bootstrap `.agent/assets/testing/conventions.md`.
   - Write `.agent/scope/test-plan.md` detailing scenarios, table-driven cases, edge cases, and mocks.
   - Dispatch an isolated worker subagent with local environment access.
   - Execute atomic test steps in sequence. Edit test files and fixtures only.
   - If a production bug is detected, halt immediately with status `BLOCKED: PRODUCTION_BUG_DETECTED` and escalate.
   - Log step execution in `.agent/scope/run/test-progress.md` (maximum 3 attempts per step).
   - Exit with status `COMPLETED` or `BLOCKED`.

5. **Test audit and mutation verification (`test-verifier`)**:
   - Dispatch an isolated auditor worker with local environment access.
   - Check file boundaries: confirm zero modifications to production source code.
   - Run repository tool suites specified in `.agent/assets/testing/conventions.md`.
   - Execute synthetic mutation red-green check: temporarily perturb production logic, confirm test fails RED, revert perturbation, and confirm test passes GREEN.
   - Record evaluation round in `.agent/scope/run/test-verdict.md`.
   - If a genuine production defect is confirmed, mark verdict as `BLOCKED: PRODUCTION_BUG_DETECTED` and halt immediately.
   - If round count < 3 and test audit fails, return actionable findings to `tester`.
   - If round count >= 3 and checks fail, mark `BLOCKED` and escalate.
   - If verdict is `PASS`, proceed to closure.

6. **Closure and handoff (`closure`)**:
   - Confirm all required verdicts are `PASS`.
   - Verify and synchronize living assets declared in `.agent/scope/closure.md`.
   - Validate modified asset files against `.agent/skills/unslop/SKILL.md`.
   - Force-stage `.agent/scope/` and stage all changes (`git add -f .agent/scope/ && git add .`).
   - Create an unslop-compliant conventional commit on `task/<task-id>`.
   - Prune the task worktree (`git worktree remove .worktrees/<task-id>`).
   - Report the commit hash and branch handoff instructions to the user for manual review and merge.

## Resume protocol

When starting or reviving a session, inspect the filesystem to determine active state:

| Condition | Inferred phase | Action |
| :--- | :--- | :--- |
| `AGENT.md` is missing or `.agent/assets/` is empty | `UNINITIALIZED` | Run `index` to bootstrap assets and root runbook. |
| `.agent/scope/` is empty or missing | `IDLE` | Await user request, then run `planner`. |
| `coder-plan.md` exists, `run/progress.md` is missing or `IN_PROGRESS` | `IMPLEMENTATION` | Spawn `coder` to resume at the first incomplete step. |
| `run/progress.md` is `BLOCKED` | `BLOCKED` | Present blocker hypothesis from `progress.md` to user. Discuss with `planner`. |
| `run/progress.md` is `COMPLETED`, `run/verdict.md` missing or `FAIL` | `VERIFICATION` / `FIX` | If verdict is missing, run `code-verifier`. If verdict is `FAIL`, spawn `coder` on actionable findings. |
| `run/verdict.md` is `PASS`, `test-plan.md` exists, `run/test-progress.md` missing or `IN_PROGRESS` | `TEST_AUTHORING` | Spawn `tester` worker to resume at first incomplete test step. |
| `run/test-progress.md` is `BLOCKED: PRODUCTION_BUG_DETECTED` | `PRODUCTION_BUG` | Present bug reproduction details to user. Discuss with `planner`. |
| `run/test-progress.md` is `BLOCKED` | `BLOCKED` | Present test authoring blocker hypothesis to user. |
| `run/test-progress.md` is `COMPLETED`, `run/test-verdict.md` missing or `FAIL` | `TEST_AUDIT` / `TEST_FIX` | If test-verdict is missing, run `test-verifier`. If `FAIL`, spawn `tester` on actionable findings. |
| `run/test-verdict.md` is `BLOCKED: PRODUCTION_BUG_DETECTED` | `PRODUCTION_BUG` | Present defect audit findings to user. Discuss with `planner`. |
| `run/test-verdict.md` is `PASS` (or `run/verdict.md` is `PASS` if testing deferred) | `CLOSURE` | Run `closure` to sync assets, commit, and prune worktree. |
| `run/verdict.md` or `run/test-verdict.md` is `BLOCKED` | `BLOCKED` | Present audit findings from verdict files to user. Discuss with `planner`. |

## Core invariants

- Features must not add new unit test files. Testing is handled exclusively by dedicated testing agents.
- Testers must never modify production source code. If production code fails a valid test, escalate immediately.
- Agents must never audit their own diffs. Audits require isolated verifiers (`code-verifier` and `test-verifier`).
- Newly created tests must prove regression detection through synthetic mutation checks (failing RED before GREEN).
- File modifications must be surgical. Avoid full-file overwrites unless creating new files.
- The repository git status must remain clean of untracked scratch files outside `.agent/scope/`.
- Tasks executing in worktrees must never merge into main automatically. Merges remain user-controlled.
- Code comments must be functional and unslop-compliant. Never write noisy comments that merely narrate or restate code.

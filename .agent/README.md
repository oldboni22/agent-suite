# Agent framework

Universal portable framework for autonomous agent planning, implementation, verification, and knowledge management.

## Philosophy

- Zero guessing: grill the user until all architectural boundaries, constraints, and dependencies are settled.
- Strict role separation: planners plan, coders code, and verifiers audit. Agents never self-audit their own diffs.
- Ephemeral task scope: task execution state lives strictly in `.agent/scope/` and cleans up at completion.
- Persistent living assets: architectural boundaries, architectural decisions (why), and language idioms live in `.agent/assets/`.
- Style source of truth: all output adheres to the concise, plain-language rules in `.agent/skills/unslop/SKILL.md`.

## Directory layout

```text
.agent/
  README.md            # Portable framework specification (this document)
  skills/              # Reusable agent procedures and roles
    architecture/      # Maps boundaries and layers in .agent/assets/architecture/
    coder/             # Spawns client subagent to execute coder-plan.md
    grill-me/          # Relentless design tree interrogation
    language/          # Curates language idioms in .agent/assets/language/
    note/              # Captures mid-flight user hints into .agent/scope/notes.md
    planner/           # Discovers context, grills, and creates scope contracts
    unslop/            # Writing style and plain-language rules
    verifier/          # Spawns client subagent to audit working tree diffs
    why/               # Records architectural decisions in .agent/assets/why/
  assets/              # Long-lived repository knowledge base
    architecture/      # System architecture and module boundaries
    language/          # Language-specific conventions and LLM blind spots
    why/               # Architectural decisions, trade-offs, and invariants
  scope/               # Transient active task workspace (git-ignored)
    intro.md           # Task definition, scope boundary, and invariants
    coder-plan.md      # Atomic step-by-step implementation plan
    closure.md         # Documentation sync instructions and cleanup checklist
    notes.md           # Mid-flight user hints and constraints
    run/               # Execution audit trail
      progress.md      # Step execution log and iteration counter (max 3 attempts)
      verdict.md       # Multi-round audit verdict (boundary, invariants, language, tests)
```

## Task lifecycle state machine

A task moves through deterministic phases:

1. **Discovery and planning (`planner`)**:
   - Inspect `.agent/assets/` (`architecture/`, `why/`, `language/`).
   - Run `grill-me` to resolve design choices, boundaries, and trade-offs.
   - Emit `.agent/scope/intro.md`, `.agent/scope/coder-plan.md`, and `.agent/scope/closure.md`.

2. **Implementation (`coder`)**:
   - Spawn client subagent (`runInClient: true`).
   - Ingest `coder-plan.md` and relevant `language/<lang>.md`.
   - Execute atomic steps in sequence. Perform surgical, in-place edits.
   - Run verification command after each step.
   - Log iteration attempts in `.agent/scope/run/progress.md` (maximum 3 attempts per step).
   - Never create new unit test files or test functions during feature work.
   - Exit with status `COMPLETED` or `BLOCKED`. Never self-audit diffs.

3. **Audit and verification (`verifier`)**:
   - Spawn fresh client subagent (`runInClient: true`).
   - Inspect git working tree deltas via `git status` and `git diff`.
   - Evaluate against 5 check categories: Scope boundaries, Invariants, Language conventions, Testing policy, and Build/existing tests.
   - Record evaluation round in `.agent/scope/run/verdict.md`.
   - If verdict is `FAIL`, document actionable findings (file, line, broken rule) and return to implementation.
   - If verdict is `PASS`, proceed to closure.

4. **Closure and handoff**:
   - Execute asset sync tasks listed in `.agent/scope/closure.md` (update architecture, why, or language docs).
   - Clean up or archive `.agent/scope/`.

## Resume protocol

When starting or reviving a session, inspect the filesystem to determine active state:

| Condition | Inferred phase | Action |
| :--- | :--- | :--- |
| `.agent/scope/` is empty or missing | `IDLE` | Await user request, then run `planner`. |
| `coder-plan.md` exists, `run/progress.md` is missing or `IN_PROGRESS` | `IMPLEMENTATION` | Spawn `coder` to resume at the first incomplete step. |
| `run/progress.md` is `BLOCKED` | `BLOCKED` | Present blocker hypothesis from `progress.md` to user. Discuss with `planner`. |
| `run/progress.md` is `COMPLETED`, `run/verdict.md` missing or `FAIL` | `VERIFICATION` / `FIX` | If verdict is missing, run `verifier`. If verdict is `FAIL`, spawn `coder` on actionable findings. |
| `run/verdict.md` is `PASS` | `CLOSURE` | Update assets per `closure.md` and retire `.agent/scope/`. |

## Core invariants

- Features must not add new unit test files. Testing is handled by dedicated testing agents.
- Agents must never audit their own diffs. Audits require an isolated verifier.
- File modifications must be surgical. Avoid full-file overwrites unless creating new files.
- The repository git status must remain clean of untracked scratch files outside `.agent/scope/`.

---
name: coder
description: Execute scoped implementation plans from .agent/scope/coder-plan.md by spawning a dedicated client subagent to perform surgical code changes, run step-level verification, and log progress in .agent/scope/run/progress.md.
disable-model-invocation: true
---

# Coder

Execute implementation plans produced by the planner. The coder orchestrator spawns a dedicated subagent running in the client environment. The subagent reads the plan and relevant language assets, performs surgical in-place edits, enforces step-level verification gates, and records an execution audit log without auditing its own diff or inventing new unit tests.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Orchestrator workflow

1. Inspect `.agent/scope/coder-plan.md` to verify the execution plan exists.
2. Read the relevant language guide asset in `.agent/assets/language/<lang>.md`.
3. Spawn a dedicated subagent using `callSubAgent` with `runInClient: true`.
4. Inject into the subagent prompt:
   - Full content of `.agent/scope/coder-plan.md`.
   - Relevant content of `.agent/assets/language/<lang>.md`.
   - The worker execution rules listed below.
5. Await subagent completion. Inspect `.agent/scope/run/progress.md` to determine if implementation completed or blocked.

## Worker execution rules

The subagent must adhere to these rules:

1. **Working directory and isolation**:
   - Operate strictly within the current repository.
   - Initialize `.agent/scope/run/progress.md` before executing steps.

2. **Sequential execution**:
   - Execute atomic steps from `coder-plan.md` in strict numerical order.
   - Do not skip steps or bundle multiple steps together.

3. **Surgical edits**:
   - Read the target file first before modifying it. Understand surrounding structure, indentation, and imports.
   - Perform precise in-place updates. Do not overwrite entire files unless creating a brand new file.
   - Preserve existing unrelated logic and comments.

4. **Testing policy**:
   - Do not create new unit test files.
   - Do not add new unit tests to existing test files.
   - If existing tests fail due to intentional contract changes specified in the plan, update the assertions to match the new contract.

5. **Step verification gate**:
   - Run the specified verification command immediately after completing changes for that step.
   - If verification passes, log the step status as PASS in `.agent/scope/run/progress.md`. Proceed to the next step.
   - If verification fails, increment the iteration counter for that step in `.agent/scope/run/progress.md`. Attempt a targeted fix within the bounds of that step.

6. **Iteration limit and blocking**:
   - A step allows a maximum of 3 attempts (1 initial attempt plus 2 retries).
   - If a step fails verification on the 3rd attempt, halt execution immediately.
   - Mark overall status as BLOCKED in `.agent/scope/run/progress.md`.
   - Record the blocker hypothesis and exact failure output. Exit without modifying additional files.

7. **Completion and handoff**:
   - When all steps pass verification, mark overall status as COMPLETED in `.agent/scope/run/progress.md`.
   - Do not attempt to self-audit diffs or clean up `.agent/scope/`.
   - Exit cleanly and report status to the orchestrator.

---

## Template for progress.md

```markdown
# Implementation progress

## Overall status: <IN_PROGRESS | BLOCKED | COMPLETED>

## Step 1: <step title>
- Status: <PASS | FAIL>
- Iterations: 1
- Notes: <optional observations or adjustments>

## Step 2: <step title>
- Status: <PASS | FAIL | BLOCKED>
- Iterations: <count, max 3>
- Iteration 1: <PASS or failure details>
- Iteration 2: <PASS or failure details>
- Iteration 3: <failure details if reached limit>
- Blocker hypothesis: <required only if status is BLOCKED>
```

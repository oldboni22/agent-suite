---
name: tester
description: Plan test scenarios into .agent/scope/test-plan.md and execute test authoring by spawning a dedicated client subagent to write tests, run step-level verification, and log progress in .agent/scope/run/test-progress.md.
disable-model-invocation: true
---

# Tester

Plan test suites and execute test authoring. The tester orchestrator inspects code contracts, bootstraps or reads repository testing conventions from `.agent/assets/testing/`, drafts `.agent/scope/test-plan.md`, and spawns an isolated worker subagent in the client environment. The worker implements tests step-by-step, enforces step-level verification gates, and logs execution progress in `.agent/scope/run/test-progress.md` without modifying production source code.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Orchestrator workflow

1. Inspect `.agent/scope/intro.md` or git diffs of the target feature branch to identify modified behaviors and contracts.
2. Check if `.agent/assets/testing/conventions.md` exists.
   - If missing, discover the repository test runner, assertion libraries, and execution commands from package manifests and build configs.
   - If conventions remain ambiguous, run the `grill-me` workflow with the user to resolve them.
   - Write `.agent/assets/testing/conventions.md`.
3. Create `.agent/scope/test-plan.md` detailing test scenarios, table-driven test cases, edge cases, and mocking boundaries using the template below.
4. Dispatch an isolated execution worker with local shell and filesystem access:
   - Host platforms with subagent tools (e.g. LobeHub): call the subagent tool with client environment execution (`runInClient: true`).
   - Single-agent harnesses or CLI tools (e.g. Claude Code, Cursor, Aider): run an isolated implementation session with a clean context fence.
5. Inject into the subagent prompt:
   - Full content of `.agent/scope/test-plan.md`.
   - Relevant content of `.agent/assets/testing/conventions.md`.
   - Relevant content of `.agent/assets/language/<lang>.md`.
   - The worker execution rules listed below.
6. Await subagent completion. Inspect `.agent/scope/run/test-progress.md` to determine if test authoring completed, blocked, or exposed a production bug.

## Worker execution rules

The subagent must adhere to these rules:

1. **Working directory and isolation**:
   - Operate strictly within the current repository.
   - Initialize `.agent/scope/run/test-progress.md` before executing steps.

2. **Strict file boundary**:
   - Create or edit test files, test fixtures, and test mocks only.
   - Do not modify production source files under any circumstances.

3. **Production defect detection and immediate escalation**:
   - If an existing production function fails a valid test case that reflects verified domain rules or API contracts, halt execution immediately.
   - Do not patch or alter production code to force tests to pass.
   - Mark overall status as `BLOCKED: PRODUCTION_BUG_DETECTED` in `.agent/scope/run/test-progress.md`.
   - Record the exact failure output, reproduction command, and root-cause hypothesis. Exit immediately to allow the planner and user to resolve the bug.

4. **Sequential execution**:
   - Execute atomic steps from `test-plan.md` in strict numerical order.
   - Do not skip steps or bundle multiple steps together.

5. **Surgical edits and idiomatic structure**:
   - Read existing test files first to match local testing patterns, naming conventions, and assertions.
   - Perform precise in-place updates when augmenting existing test suites.

6. **Code comments**:
   - Comment only complex fixtures, synthetic data generators, or non-obvious mock arrangements.
   - Do not write narrative comments that restate assertion logic.
   - Follow unslop rules strictly: plain words, active voice, sentence case, straight quotes, no decorative emojis, and no em dashes.

7. **Step verification gate**:
   - Run the test verification command specified in `.agent/assets/testing/conventions.md` after each step.
   - If verification passes, log the step status as PASS in `.agent/scope/run/test-progress.md`. Proceed to the next step.
   - If verification fails due to a flaw in test code or mock setup, increment the iteration counter in `.agent/scope/run/test-progress.md`. Attempt a targeted fix within that step.

8. **Iteration limit and blocking**:
   - A step allows a maximum of 3 attempts (1 initial attempt plus 2 retries).
   - If a step fails test execution on the 3rd attempt, halt execution immediately.
   - Mark overall status as BLOCKED in `.agent/scope/run/test-progress.md`.
   - Record the blocker hypothesis and exact failure output. Exit cleanly.

9. **Completion and handoff**:
   - When all steps pass verification, mark overall status as COMPLETED in `.agent/scope/run/test-progress.md`.
   - Do not attempt to self-audit diffs or run mutation audits. Auditing belongs strictly to `test-verifier`.
   - Exit cleanly and report status to the orchestrator.

---

## Template for test-plan.md

```markdown
# Test plan: <short title>

## Constraints and boundaries
- **File boundaries**: Create and edit test files, mocks, and fixtures only. Modifying production code is strictly forbidden.
- **Escalation policy**: Halt immediately with status BLOCKED: PRODUCTION_BUG_DETECTED if a production bug is encountered.

## Pre-flight context
- [ ] Read .agent/assets/testing/conventions.md for repository test tools and conventions.
- [ ] Read relevant language guide in .agent/assets/language/<lang>.md.
- [ ] Confirm target contracts and diffs from feature work.

## Test execution steps

### Step 1: <test step title>
- **Target test file**: `<path>`
- **Scenarios to cover**:
  - Happy path: <expected input and output>
  - Error condition: <input causing error and expected error type or code>
  - Edge case: <boundary input value>
- **Conventions to follow**: <specific rule from testing conventions>
- **Verification command**: `<test command>`

### Step 2: <test step title>
- **Target test file**: `<path>`
- **Scenarios to cover**:
  - Happy path: <expected input and output>
  - Error condition: <input causing error and expected error type or code>
- **Conventions to follow**: <specific rule from testing conventions>
- **Verification command**: `<test command>`

## Verification rules
- Run the verification command after each step.
- Do not proceed to the next step if verification fails.
- Never edit production files.
```

---

## Template for test-progress.md

```markdown
# Test authoring progress

## Overall status: <IN_PROGRESS | BLOCKED | BLOCKED: PRODUCTION_BUG_DETECTED | COMPLETED>

## Step 1: <step title>
- Status: <PASS | FAIL>
- Iterations: 1
- Notes: <optional observations or adjustments>

## Step 2: <step title>
- Status: <PASS | FAIL | BLOCKED | BLOCKED: PRODUCTION_BUG_DETECTED>
- Iterations: <count, max 3>
- Iteration 1: <PASS or failure details>
- Iteration 2: <PASS or failure details>
- Iteration 3: <failure details if reached limit>
- Blocker hypothesis: <required if status is BLOCKED or BLOCKED: PRODUCTION_BUG_DETECTED>
```

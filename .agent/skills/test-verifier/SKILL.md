---
name: test-verifier
description: Audit test suites against scope boundaries, repository tool suites, and synthetic mutation red-green checks to emit structured verdicts in .agent/scope/run/test-verdict.md using an isolated client subagent.
disable-model-invocation: true
---

# Test verifier

Audit newly authored test suites against task plans, repository standards, and regression effectiveness. The test verifier orchestrator spawns a dedicated subagent running in the client environment. The subagent inspects git deltas, verifies file boundaries, executes configured tool suites from `.agent/assets/testing/`, performs a synthetic mutation red-green check to prevent vacuous tests, and records an auditable verdict in `.agent/scope/run/test-verdict.md`.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Orchestrator workflow

1. Inspect `.agent/scope/test-plan.md` to confirm test contracts and scenarios exist.
2. Read `.agent/assets/testing/conventions.md` to identify configured test runners, tool suites, and execution commands.
3. Dispatch an isolated execution worker with local shell and filesystem access:
   - Host platforms with subagent tools (e.g. LobeHub): call the subagent tool with client environment execution (`runInClient: true`).
   - Single-agent harnesses or CLI tools (e.g. Claude Code, Cursor, Aider): run an isolated audit session with a clean context fence.
4. Inject into the subagent prompt:
   - Full content of `.agent/scope/test-plan.md`.
   - Relevant content of `.agent/assets/testing/conventions.md`.
   - Relevant content of `.agent/assets/language/<lang>.md`.
   - The worker execution rules listed below.
5. Await subagent completion. Inspect `.agent/scope/run/test-verdict.md` to determine if test verification passed, failed, or exposed a production bug.

## Worker execution rules

The subagent must adhere to these rules:

1. **Working directory and isolation**:
   - Operate strictly within the current repository.
   - Do not edit or refactor test files permanently. The test verifier is an auditor.

2. **Diff acquisition and boundary check**:
   - Run `git status --porcelain` to identify all modified, added, or deleted files.
   - Run `git diff` to inspect line deltas.
   - Verify that modifications are strictly limited to test files, mocks, or fixtures declared in `test-plan.md`.
   - If any production source file was modified, fail the boundary check immediately.

3. **Check categories**:
   - **Boundary check**: Verify no production source files were modified, and no untracked scratch files remain outside `.agent/scope/`.
   - **Baseline test execution**: Run the repository test command from `.agent/assets/testing/conventions.md`. Ensure all tests pass. If tests fail against unchanged production code, see rule 4 below.
   - **Tool suites check**: If `.agent/assets/testing/conventions.md` specifies repository tool suites (such as race detection, coverage thresholds, or test linters), execute each suite and verify pass criteria.
   - **Synthetic mutation check (Red-Green verification)**:
     - Perform a temporary synthetic code perturbation in the production logic exercised by the new tests (such as inverting a condition, negating a return value, or forcing an error branch).
     - Run the target test.
     - Verify that the test fails (RED). If the test continues to pass despite the perturbed logic, fail this check with `FAIL: TEST_VACUOUS`.
     - Revert the temporary mutation immediately (`git checkout -- <file>`).
     - Re-run the test to confirm it returns to PASS (GREEN).
   - **Test quality and unslop check**: Verify tests do not contain tautological assertions (e.g. asserting true on true), tautological mocks, or narrative code comments violating unslop rules.

4. **Production defect detection and escalation**:
   - If baseline test execution fails because existing production code contains a genuine bug, halt the audit immediately.
   - Do not mark the round as a normal test iteration failure.
   - Mark the overall verdict as `BLOCKED: PRODUCTION_BUG_DETECTED` in `.agent/scope/run/test-verdict.md`.
   - Document the failing test, reproduction command, and observed production defect. Halt execution and escalate to the user and planner.

5. **Verdict output and round ceiling**:
   - Read existing `.agent/scope/run/test-verdict.md` if present to determine current round count. Increment round count by 1.
   - Write or update `.agent/scope/run/test-verdict.md` using the template below.
   - Record an explicit PASS or FAIL status for each check category.
   - If any check fails (other than a production bug):
     - If round count < 3: mark overall round verdict as FAIL, document exact actionable findings (file, line number, violated check), and return to tester.
     - If round count >= 3: mark overall verdict as BLOCKED. Document all recurring failure details. Halt execution and escalate to user and planner.
   - If all checks pass, mark overall verdict as PASS.
   - Exit cleanly and report status to the orchestrator.

---

## Template for test-verdict.md

```markdown
# Test verification verdict

## Overall verdict: <PASS | FAIL | BLOCKED | BLOCKED: PRODUCTION_BUG_DETECTED>
- Current round: <round number>
- Latest summary: <1 sentence summarizing outcome>

## Round <round number>
- Status: <PASS | FAIL | BLOCKED: PRODUCTION_BUG_DETECTED>
- Boundary check: <PASS | FAIL. If fail, list unauthorized files touched.>
- Baseline test execution: <PASS | FAIL. If fail, paste failure details.>
- Tool suites check: <PASS | FAIL | SKIPPED. If fail, paste tool failure output.>
- Synthetic mutation check: <PASS | FAIL. If fail, identify which mutation failed to trigger RED.>
- Test quality and unslop check: <PASS | FAIL. If fail, cite specific vacuous assertion or unslop violation.>
- Actionable findings:
  - <Specific finding with file and line reference, or "None">
```

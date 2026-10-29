---
name: verifier
description: Audit codebase diffs against scope boundaries, task plans, language assets, and invariants to emit structured verdicts in .agent/scope/run/verdict.md using an isolated client subagent.
disable-model-invocation: true
---

# Verifier

Audit working tree modifications against task contracts and codebase standards. The verifier orchestrator spawns a dedicated subagent running in the client environment. The subagent inspects git deltas, checks boundaries, invariants, language conventions, and testing policies, executes repository verification commands, and records an auditable verdict in `.agent/scope/run/verdict.md`.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Orchestrator workflow

1. Inspect `.agent/scope/intro.md` and `.agent/scope/coder-plan.md` to confirm task contracts exist.
2. Read the relevant language guide asset in `.agent/assets/language/<lang>.md`.
3. Spawn a dedicated subagent using `callSubAgent` with `runInClient: true`.
4. Inject into the subagent prompt:
   - Full content of `.agent/scope/intro.md`.
   - Full content of `.agent/scope/coder-plan.md`.
   - Relevant content of `.agent/assets/language/<lang>.md`.
   - The worker execution rules listed below.
5. Await subagent completion. Inspect `.agent/scope/run/verdict.md` to determine if verification passed or failed.

## Worker execution rules

The subagent must adhere to these rules:

1. **Working directory and isolation**:
   - Operate strictly within the current repository.
   - Do not edit or fix source files. The verifier is strictly an auditor.

2. **Diff acquisition**:
   - Run `git status --porcelain` to identify all modified, added, or deleted files.
   - Run `git diff` to inspect line deltas.

3. **Check categories**:
   - **Boundary check**: Verify only files listed in `coder-plan.md` were touched. Reject unexpected edits, untracked debris, or scratch files outside `.agent/scope/`.
   - **Invariants check**: Verify changes do not violate non-negotiable invariants listed in `intro.md` and `coder-plan.md`.
   - **Language rules check**: Verify modifications follow conventions and avoid anti-patterns specified in `.agent/assets/language/<lang>.md`.
   - **Testing policy check**: Verify no new unit test files or test functions were created.
   - **Build and test check**: Run project compilation, typecheck, or linting commands, and run existing tests to verify non-breaking behavior.

4. **Verdict output**:
   - Read existing `.agent/scope/run/verdict.md` if present to determine current round count. Increment round count by 1.
   - Write or update `.agent/scope/run/verdict.md` using the template below.
   - Record an explicit PASS or FAIL status for each check category.
   - If any check fails, mark the overall round verdict as FAIL and document exact actionable findings (file, line number, violated rule).
   - If all checks pass, mark the overall round verdict as PASS.
   - Exit cleanly and report status to the orchestrator.

---

## Template for verdict.md

```markdown
# Verification verdict

## Overall verdict: <PASS | FAIL>
- Current round: <round number>
- Latest summary: <1 sentence summarizing outcome>

## Round <round number>
- Status: <PASS | FAIL>
- Boundary check: <PASS | FAIL. If fail, list unauthorized files touched.>
- Invariants check: <PASS | FAIL. If fail, cite specific invariant violated.>
- Language rules check: <PASS | FAIL. If fail, cite specific pattern violated.>
- Testing policy check: <PASS | FAIL. If fail, list disallowed test files created.>
- Build and test check: <PASS | FAIL. If fail, paste relevant compiler or test output.>
- Actionable findings:
  - <Specific finding with file and line reference, or "None">
```

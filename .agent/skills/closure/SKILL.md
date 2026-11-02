---
name: closure
description: Finalize verified tasks by syncing documentation assets, updating AGENT.md sync markers, validating style against unslop rules, committing to the task branch, and handling worktree cleanup.
disable-model-invocation: true
---

# Closure

Finalize a verified task. The closure skill executes when all required verdicts in the task pipeline reach a `PASS` status. It synchronizes living assets listed in `closure.md`, updates the `last_synced_commit` in root `AGENT.md`, validates text against unslop rules, commits all changes and task audit records to the task branch, reports the handoff to the user, and cleans up the active execution environment.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Workflow

1. **Pre-flight verification gate**:
   - Inspect `.agent/scope/intro.md` to review the active `## Pipeline`.
   - Confirm that every required verification stage in the pipeline (e.g. `code-verifier`, and `test-verifier` if present) recorded a `PASS` verdict.
   - Halt immediately if any required verdict is missing, `FAIL`, or `BLOCKED`.

2. **Asset synchronization and verification**:
   - Read `.agent/scope/closure.md`.
   - Check each declared asset update under `Documentation and assets to update`:
     - Architecture: verify corresponding files in `.agent/assets/architecture/` were created or updated.
     - Decisions (why): verify corresponding files in `.agent/assets/why/` were created or updated.
     - Language guide: verify corresponding files in `.agent/assets/language/` were created or updated.
     - Domain knowledge: verify corresponding files in `.agent/assets/domain/` were created or updated.
     - Flow: verify `.agent/assets/flow/lifecycle.md` was updated if new stages or profiles were added.
   - If any declared asset update is missing or unwritten, halt closure with an error.

3. **Unslop style gate**:
   - Run a style audit across all modified or newly created files in `.agent/assets/`.
   - Check against `.agent/skills/unslop/SKILL.md`:
     - No AI vocabulary or filler phrases (Rules 7, 23).
     - No emojis in headings or bullets (Rule 18).
     - Straight quotes only (Rule 19).
     - No em dashes or double hyphens (Rule 13).
     - Sentence case headings (Rule 17).
   - If violations exist, report the exact file, line, and rule, and halt closure until corrected.

4. **Git staging, commit, and sync marker**:
   - Force-stage the task audit scope: `git add -f .agent/scope/`.
   - Stage all source code and asset modifications: `git add .`.
   - Create a conventional commit following unslop rules:
     ```text
     <type>(<scope>): <short imperative summary in sentence case>

     Task: <task-id>
     Verified steps: <count from coder-plan.md>
     Assets updated: <list of modified asset files>
     ```
   - Obtain the resulting commit SHA via `git rev-parse HEAD`.
   - Update `last_synced_commit: <sha>` in the root `AGENT.md` (and amend or commit the marker update).

5. **Execution mode cleanup**:
   - Inspect `Execution mode` in `.agent/scope/intro.md`.
   - **In-place mode**: Keep the branch `task/<task-id>` checked out in the working directory.
   - **Worktree mode**:
     - Switch out of the worktree directory to the primary repository root.
     - Remove the task worktree: `git worktree remove .worktrees/<task-id>`.
     - Prune stale worktrees: `git worktree prune`.
     - Leave the branch `task/<task-id>` intact in git history.

6. **User handoff report**:
   - Report the task completion summary to the user:
     - Branch name and commit hash.
     - Verified steps and audit summary.
     - List of updated assets.
     - Command for the user to review and merge into main:
       ```bash
       git checkout main
       git merge task/<task-id>
       ```

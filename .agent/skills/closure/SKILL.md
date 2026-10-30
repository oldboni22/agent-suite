---
name: closure
description: Finalize verified tasks by syncing documentation assets, validating style against unslop rules, committing to the task branch, and pruning the git worktree.
disable-model-invocation: true
---

# Closure

Finalize a verified task. The closure skill executes when `.agent/scope/run/verdict.md` reaches a `PASS` status. It synchronizes living assets listed in `closure.md`, validates text against unslop rules, commits all changes and task audit records to the task branch, reports the handoff to the user, and prunes the task worktree.

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
   - Inspect `.agent/scope/run/verdict.md`.
   - Confirm that the overall verdict is `PASS`.
   - Halt immediately if the verdict is missing, `FAIL`, or `BLOCKED`.

2. **Asset synchronization and verification**:
   - Read `.agent/scope/closure.md`.
   - Check each declared asset update under `Documentation and assets to update`:
     - Architecture: verify corresponding files in `.agent/assets/architecture/` were created or updated.
     - Decisions (why): verify corresponding files in `.agent/assets/why/` were created or updated.
     - Language guide: verify corresponding files in `.agent/assets/language/` were created or updated.
     - Domain knowledge: verify corresponding files in `.agent/assets/domain/` were created or updated.
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

4. **Git staging and commit**:
   - Force-stage the task audit scope: `git add -f .agent/scope/`.
   - Stage all source code and asset modifications: `git add .`.
   - Create a conventional commit following unslop rules:
     ```text
     <type>(<scope>): <short imperative summary in sentence case>

     Task: <task-id>
     Verified steps: <count from coder-plan.md>
     Assets updated: <list of modified asset files>
     ```
   - The task branch now permanently retains code, asset documentation, and the full audit trail.

5. **Worktree cleanup**:
   - Record the commit hash and branch name.
   - Switch out of the worktree directory to the repository root.
   - Remove the worktree: `git worktree remove .worktrees/<task-id>`.
   - Leave the branch `task/<task-id>` intact for user review.

6. **User handoff report**:
   - Report the task completion summary to the user:
     - Branch name and commit hash.
     - Verified steps and audit summary.
     - List of updated assets.
     - Command for the user to review and merge into main:
       ```bash
       git checkout main
       git merge task/<task-id>
       # Note: If desired, remove .agent/scope/ from main prior to or after merge.
       ```

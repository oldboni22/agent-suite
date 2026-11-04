# Reindexing procedure

Audit and synchronize existing framework assets against codebase evolution.

## Guiding principle

Reindexing maintains accuracy without destroying historical knowledge. The agent reads existing assets as the established baseline and scans for drift. It never overwrites confirmed invariants from scratch. The agent grills the user only on detected discrepancies and newly introduced concepts.

## Procedure

1. **Baseline asset intake**:
   - Read line 1 of `.agent/assets/index.md` (`loc: [0, 1]`) to retrieve `last_synced_commit: <sha>`.
   - Read `.agent/assets/architecture/system.md` and module documents.
   - Read `.agent/assets/domain/general.md` and topic documents.
   - Read `.agent/assets/language/<lang>.md` and `.agent/assets/testing/conventions.md`.
   - Initialize `.agent/scope/index-state.md` with current asset summaries.

2. **Drift detection**:
   - Run `git log <sha>..HEAD --oneline` on source directories to isolate unindexed commits.
   - Compare the current codebase file tree and package manifests against existing architecture and language assets.
   - Identify structural drift: new directories, deleted packages, relocated entry points, or altered module boundaries.
   - Identify domain drift: newly added public types, altered state machines, or new external API endpoints not documented in domain assets.
   - Identify tooling drift: updated test frameworks, new linters, changed build scripts, or revised CLI flags.

3. **Targeted user interview**:
   - Do not re-ask previously confirmed questions.
   - Run the `grill-me` workflow strictly on newly detected drift, complying with `honest` rules:
     - Clarify the business role and boundaries of new packages.
     - Confirm domain rules and invariants for newly introduced entities or workflows.
     - Confirm updated build or test commands.

4. **Surgical asset updates**:
   - Update `.agent/assets/architecture/system.md` and create or revise `.agent/assets/architecture/modules/<module-name>.md` for new components.
   - Update `.agent/assets/domain/general.md` or create new `.agent/assets/domain/topics/<topic>.md` files for new business areas.
   - Update `.agent/assets/testing/conventions.md` with new test commands or tool suites.
   - Update `.agent/assets/language/<lang>.md` if new idioms or guardrails were established.
   - Preserve existing invariant records unless the user explicitly requested a change.

5. **Asset index synchronization**:
   - Update `.agent/assets/index.md`.
   - Ensure line 1 contains the new HEAD SHA: `last_synced_commit: <git rev-parse HEAD>`.
   - Update canonical CLI commands and active asset pointers if modified.
   - Do not touch user-owned root files (`AGENT.md`, `CLAUDE.md`).

6. **Style audit**:
   - Validate all modified assets against `.agent/skills/unslop/SKILL.md` and `.agent/skills/honest/SKILL.md`.
   - Check that text uses active voice, plain words, sentence case headings, straight quotes, and zero em dashes or decorative emojis.

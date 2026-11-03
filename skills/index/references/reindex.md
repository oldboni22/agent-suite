# Reindexing procedure

Audit and synchronize existing framework assets against codebase evolution.

## Guiding principle

Reindexing maintains accuracy without destroying historical knowledge. The agent reads existing assets as the established baseline and scans for drift. It never overwrites confirmed invariants from scratch. The agent grills the user only on detected discrepancies and newly introduced concepts.

## Procedure

1. **Baseline asset intake**:
   - Read `.agent/assets/architecture/system.md` and module documents.
   - Read `.agent/assets/domain/general.md` and topic documents.
   - Read `.agent/assets/language/<lang>.md` and `.agent/assets/testing/conventions.md`.
   - Read the existing root `AGENT.md`.
   - Initialize `.agent/scope/index-state.md` with current asset summaries.

2. **Drift detection**:
   - Compare the current codebase file tree and package manifests against existing architecture and language assets.
   - Identify structural drift: new directories, deleted packages, relocated entry points, or altered module boundaries.
   - Identify domain drift: newly added public types, altered state machines, or new external API endpoints not documented in domain assets.
   - Identify tooling drift: updated test frameworks, new linters, changed build scripts, or revised CLI flags.

3. **Targeted user interview**:
   - Do not re-ask previously confirmed questions.
   - Run the `grill-me` workflow strictly on newly detected drift:
     - Clarify the business role and boundaries of new packages.
     - Confirm domain rules and invariants for newly introduced entities or workflows.
     - Confirm updated build or test commands.

4. **Surgical asset updates**:
   - Update `.agent/assets/architecture/system.md` and create or revise `.agent/assets/architecture/modules/<module-name>.md` for new components.
   - Update `.agent/assets/domain/general.md` or create new `.agent/assets/domain/topics/<topic>.md` files for new business areas.
   - Update `.agent/assets/testing/conventions.md` with new test commands or tool suites.
   - Update `.agent/assets/language/<lang>.md` if new idioms or guardrails were established.
   - Preserve existing invariant records unless the user explicitly requested a change.

5. **Root runbook maintenance**:
   - Review `AGENT.md` at the repository root.
   - Update execution commands or working directory notes if build configurations changed.
   - Verify that `AGENT.md` continues to point to `.agent/README.md` and contains zero duplications of framework procedures or asset details.

6. **Style audit**:
   - Validate all modified assets against `.agent/skills/unslop/SKILL.md`.
   - Check that text uses active voice, plain words, sentence case headings, straight quotes, and zero em dashes or decorative emojis.

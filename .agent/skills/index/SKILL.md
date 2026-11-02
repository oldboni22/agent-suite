---
name: index
description: Initialize and maintain repository living assets in .agent/assets/ and the root AGENT.md by inspecting codebase state, tracking last synced commits, and grilling the user via dedicated path references.
disable-model-invocation: true
---

# Index

Initialize and maintain repository living assets in `.agent/assets/` and the root operational runbook in `AGENT.md`. The index skill detects repository maturity, selects the corresponding onboarding or maintenance reference guide, tracks exploration progress in `.agent/scope/index-state.md`, verifies the lifecycle flow in `.agent/assets/flow/lifecycle.md`, tracks commit sync markers in `AGENT.md`, writes directly to asset files, and eliminates ambiguity through relentless user grilling.

## Universal skills and repository assets invariant

Maintain strict boundary separation between framework skills and living assets:
- Framework skills in `.agent/skills/` are universal, portable, and tool-agnostic. They never hardcode specific programming languages, build runners, or test commands.
- Living assets in `.agent/assets/` are repository-specific and concrete. They store accumulated project knowledge, exact CLI commands, module boundaries, domain rules, and testing conventions.

## Root AGENT.md maintenance rules

The root `AGENT.md` is the primary entry point and operational runbook for external agents contacting the repository:
- Locate `AGENT.md` strictly at the repository root, never inside `.agent/`.
- Point to `.agent/README.md` as the authoritative driver for framework lifecycle, role state machines, and core invariants.
- Track asset synchronization freshness: maintain `last_synced_commit: <sha>` near the top of `AGENT.md`.
- State repository operational constraints: working directory, execution paths, unslop compliance, and zero-guessing rules.
- List canonical operational commands: compile, typecheck, lint, test, and run.
- Never duplicate framework lifecycle tables, skill rosters, or full architectural specifications. Keep `AGENT.md` lean and factual.

## Style and writing rules

All output produced by this skill and all generated asset files must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Workflow

1. **Path selection**:
   Inspect the working tree and `.agent/assets/` to determine the repository condition:
   - **Reindex path**: If `.agent/assets/architecture/system.md` or `.agent/assets/domain/general.md` exists, load `.agent/skills/index/references/reindex.md`. Check `last_synced_commit` in `AGENT.md` and run `git log <sha>..HEAD --oneline` on `src/` to identify unindexed human commits.
   - **Greenfield path**: If the repository contains no source code or package manifests (empty or only git metadata), load `.agent/skills/index/references/greenfield.md`.
   - **Brownfield path**: If source files and manifests exist but living assets are missing, load `.agent/skills/index/references/brownfield.md`.

2. **State ledger tracking**:
   - For multi-directory exploration, initialize `.agent/scope/index-state.md` using the template below.
   - Record visited paths, candidate modules, settled domain concepts, and pending user questions.
   - Write confirmed findings directly to `.agent/assets/` as discovery progresses. Living assets serve as the permanent index state.

3. **Lifecycle flow verification**:
   - Verify that `.agent/assets/flow/lifecycle.md` exists and defines the repository lifecycle pipeline.
   - If missing, run the `flow` skill workflow to establish repository defaults based on testing conventions.

4. **User interview gate**:
   - Run the `grill-me` skill workflow sequentially for all ambiguous boundaries, business invariants, and entity lifecycles.
   - Finding technical facts is the agent's job. Explaining business rules, external partner guarantees, and architectural intent belongs to the user.
   - Never extrapolate unproven business logic from code naming alone.

5. **Asset synthesis and validation**:
   - Verify all generated or updated assets match templates defined in corresponding skills (`architecture`, `domain`, `language`, `testing`, `why`, `flow`).
   - Run the unslop style audit across all generated files before concluding.
   - Update or verify the root `AGENT.md`, recording `last_synced_commit: <sha>` using current `git rev-parse HEAD`.

---

## Template for index-state.md

```markdown
# Indexing state

## Repository condition: <GREENFIELD | BROWNFIELD | REINDEX>

## Explored paths
- `<path>`: <summary of contents and component type>

## Settled assets
- `<path>`: <status: CREATED | UPDATED | VERIFIED>

## Open questions for user
- <Question or trade-off to resolve via grill-me>
```

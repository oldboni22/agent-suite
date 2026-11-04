---
name: index
description: Initialize and maintain repository living assets in .agent/assets/ and .agent/assets/index.md by inspecting codebase state, tracking last synced commits, and grilling the user via dedicated path references.
disable-model-invocation: true
---

# Index

Initialize and maintain repository living assets in `.agent/assets/` and the entry manifest in `.agent/assets/index.md`. The index skill detects repository maturity, selects the corresponding onboarding or maintenance reference guide, tracks exploration progress in `.agent/scope/index-state.md`, verifies commit sync markers in line 1 of `.agent/assets/index.md`, writes directly to asset files, and eliminates ambiguity through relentless user grilling.

## Universal skills and repository assets invariant

Maintain strict boundary separation between framework skills and living assets:
- Framework skills in `.agent/skills/` are universal, portable, and tool-agnostic. They never hardcode specific programming languages, build runners, or test commands.
- Living assets in `.agent/assets/` are repository-specific and concrete. They store accumulated project knowledge, exact CLI commands, module boundaries, domain rules, and testing conventions.
- Living assets are completely self-contained. The repository root (`AGENT.md`, `CLAUDE.md`, or custom user manifests) is fully owned by the user and decoupled from the asset memory store.

## Asset index (.agent/assets/index.md) maintenance rules

`.agent/assets/index.md` is the primary entry point and machine-readable manifest for agents exploring repository assets:
- **First line commit sync marker**: Line 1 MUST strictly be `last_synced_commit: <sha>`. Tools capable of reading single line ranges (`loc: [0, 1]`) read this line directly to verify freshness without parsing the entire file.
- **Asset manifest**: Provide direct paths to active living assets (`architecture/system.md`, `domain/general.md`, `testing/conventions.md`, `language/<lang>.md`, `why/`).
- **Canonical execution commands**: List essential one-line CLI verification commands (build, lint, test) so agents can verify changes without traversing multiple asset subdirectories.
- Keep `.agent/assets/index.md` concise and strictly under 30 lines. Never duplicate full architecture or domain specifications.

## Style and writing rules

All output produced by this skill and all generated asset files must follow `.agent/skills/unslop/SKILL.md` and `.agent/skills/honest/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Workflow

1. **Path selection**:
   Inspect the working tree and `.agent/assets/` to determine the repository condition:
   - **Reindex path**: If `.agent/assets/index.md`, `.agent/assets/architecture/system.md`, or `.agent/assets/domain/general.md` exists, load `.agent/skills/index/references/reindex.md`. Check `last_synced_commit` on line 1 of `.agent/assets/index.md` and run `git log <sha>..HEAD --oneline` on source directories to identify unindexed commits.
   - **Greenfield path**: If the repository contains no source code or package manifests (empty or only git metadata), load `.agent/skills/index/references/greenfield.md`.
   - **Brownfield path**: If source files and manifests exist but living assets are missing, load `.agent/skills/index/references/brownfield.md`.

2. **State ledger tracking**:
   - For multi-directory exploration, initialize `.agent/scope/index-state.md` using the template below.
   - Record visited paths, candidate modules, settled domain concepts, and pending user questions.
   - Write confirmed findings directly to `.agent/assets/` as discovery progresses. Living assets serve as the permanent index state.

3. **User interview gate**:
   - Run the `grill-me` skill workflow sequentially for all ambiguous boundaries, business invariants, and entity lifecycles.
   - Comply with `honest` rules: provide objective mechanistic pushback on technical flaws without flattery or apologies.
   - Finding technical facts is the agent's job. Explaining business rules, external partner guarantees, and architectural intent belongs to the user.
   - Never extrapolate unproven business logic from code naming alone.

4. **Asset synthesis and validation**:
   - Verify all generated or updated assets match templates defined in corresponding skills (`architecture`, `domain`, `language`, `testing`, `why`).
   - Run the unslop and honest style audits across all generated files before concluding.
   - Update or generate `.agent/assets/index.md`, placing `last_synced_commit: <sha>` on line 1 using current `git rev-parse HEAD`.

---

## Template for index.md (.agent/assets/index.md)

```markdown
last_synced_commit: <git_commit_sha>

# Repository assets index

## Quick commands
- **Build**: `<canonical build command>`
- **Lint**: `<canonical lint/vet command>`
- **Test**: `<canonical test command>`

## Living assets
- **Architecture**: `.agent/assets/architecture/system.md`
- **Domain**: `.agent/assets/domain/general.md`
- **Testing**: `.agent/assets/testing/conventions.md`
- **Language**: `.agent/assets/language/<lang>.md`
- **Decisions (why)**: `.agent/assets/why/`
```

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

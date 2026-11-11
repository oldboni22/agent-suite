---
name: index
description: Initialize and maintain repository living assets in .agent/assets/ and .agent/assets/index.md by inspecting codebase state, discovering architecture and domain concepts, and grilling the user via dedicated path references.
disable-model-invocation: true
---

# Index

Initialize and maintain repository living assets in `.agent/assets/` and the entry manifest in `.agent/assets/index.md`. The index skill detects repository condition, selects the corresponding onboarding or maintenance reference guide, writes directly to asset files, and eliminates ambiguity through relentless user grilling.

## Universal skills and repository assets invariant

Maintain strict boundary separation between framework skills and living assets:
- Framework skills are universal, portable, and tool-agnostic. They never hardcode specific programming languages, build runners, or test commands.
- Living assets in `.agent/assets/` are repository-specific and concrete. They store accumulated project knowledge, exact CLI commands, module boundaries, domain rules, and testing conventions.
- Living assets are completely self-contained. The repository root (`AGENT.md`, `CLAUDE.md`, or custom user manifests) is fully owned by the user and decoupled from the asset memory store.

## Asset index (.agent/assets/index.md) maintenance rules

`.agent/assets/index.md` is the primary entry point and machine-readable manifest for agents exploring repository assets:
- **Asset manifest**: Provide direct paths to active living assets (`architecture/system.md`, `domain/general.md`, `testing/conventions.md`, `language/<lang>.md`, `why/`).
- **Canonical execution commands**: List essential one-line CLI verification commands (build, lint, test) so agents can verify changes without traversing multiple asset subdirectories.
- Keep `.agent/assets/index.md` concise and strictly under 30 lines. Never duplicate full architecture or domain specifications.

## Style and writing rules

All output produced by this skill and all generated asset files must comply with `unslop` and `honest` rules.

## Workflow

1. **Path selection**:
   Inspect the working tree and `.agent/assets/` to determine the repository condition:
   - **Reindex path**: If `.agent/assets/index.md`, `.agent/assets/architecture/system.md`, or `.agent/assets/domain/general.md` exists, load `references/reindex.md`. Check existing assets and compare against current codebase files to identify drift.
   - **Greenfield path**: If the repository contains no source code or package manifests (empty or only git metadata), load `references/greenfield.md`.
   - **Brownfield path**: If source files and manifests exist but living assets are missing, load `references/brownfield.md`.

2. **User interview gate**:
   - Run the `grill-me` skill workflow sequentially for ambiguous boundaries, business invariants, and entity lifecycles.
   - Comply with `honest` rules: provide objective mechanistic pushback on technical flaws without flattery or apologies.
   - Finding technical facts is the agent's job. Explaining business rules, external partner guarantees, and architectural intent belongs to the user.
   - Never extrapolate unproven business logic from code naming alone.

3. **Asset synthesis and validation**:
   - Verify all generated or updated assets match templates defined in corresponding skills (`architecture`, `domain`, `language`, `testing`, `why`).
   - Run the unslop and honest style audits across all generated files before concluding.
   - Update or generate `.agent/assets/index.md` with canonical commands and active asset pointers.

---

## Template for index.md (.agent/assets/index.md)

```markdown
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

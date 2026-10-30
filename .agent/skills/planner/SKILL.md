---
name: planner
description: Plan tasks and features into .agent/scope/ (intro.md, coder-plan.md, closure.md) using codebase discovery, asset inspection, and relentless grilling to eliminate ambiguity.
disable-model-invocation: true
---

# Planner

Plan features, refactors, and fixes by producing structured execution artifacts in `.agent/scope/`. The planner reads existing architecture and decision records, explores the codebase to find boundaries and redlines, grills the user until all dependencies are settled, and writes explicit step-by-step instructions for downstream agents.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Workflow

1. **Context discovery**:
   - Inspect `.agent/assets/architecture/system.md` and relevant module files to determine component boundaries.
   - Inspect `.agent/assets/why/` to identify non-negotiable invariants and past technical decisions.
   - Inspect `.agent/assets/language/<lang>.md` to identify project-specific language idioms and standards.
   - Inspect `.agent/assets/domain/general.md` and relevant topic files under `.agent/assets/domain/topics/` to identify business invariants and entity lifecycles.
   - Inspect any files or assets explicitly designated by the user.
   - Search the codebase to map call chains, imports, and touchpoint files.

2. **Grill gate**:
   - Run the `grill-me` skill workflow following the format and rules in `.agent/skills/grill-me/SKILL.md`.
   - Ask decisions sequentially, presenting one question per turn with background, explicit options, and a recommended answer.
   - If discovery shows zero ambiguity and trivial touchpoints, summarize baseline assumptions and ask the user for confirmation before writing files.

3. **Emit scope artifacts**:
   - Initialize `.agent/scope/`.
   - Write `.agent/scope/intro.md` using the template below.
   - Write `.agent/scope/coder-plan.md` using the template below.
   - Write `.agent/scope/closure.md` using the template below.

---

## Template for intro.md

```markdown
# Task: <short title>

## Problem
1 to 2 sentences stating the problem, bug, or feature request.

## Scope boundary
Explicit statement of what is included and what is excluded.
- **In scope**: <concrete deliverables and behaviors>
- **Out of scope**: <forbidden additions, non-goals, and deferred tasks>

## High-level approach
2 to 3 sentences outlining the technical path and affected subsystems.

## Invariants and constraints
Non-negotiable rules derived from .agent/assets/why/ and project architecture that must not be broken.
- <invariant 1>
- <invariant 2>
```

---

## Template for coder-plan.md

```markdown
# Coder plan: <short title>

## Constraints and invariants
- **Invariants**: <non-negotiable rules derived from .agent/assets/why/ and project architecture>
- **Out of scope**: <explicit forbidden additions, non-goals, and deferred changes>
- **Testing policy**: Do not create new unit tests. Features must not introduce new test files.

## Pre-flight context
- [ ] Read and review .agent/assets/language/<lang>.md for language idioms and conventions.
- [ ] Review relevant architecture boundaries in .agent/assets/architecture/.
- [ ] Confirm no new unit tests will be created during this feature task.

## Execution steps

### Step 1: <step title>
- **Files to modify**: `<path>`
- **Action**: <concrete change to make>
- **Language rules to follow**: <specific rule or pattern from language assets>
- **Verification command**: `<build, compile, or lint command, or existing test command>`

### Step 2: <step title>
- **Files to modify**: `<path>`
- **Action**: <concrete change to make>
- **Language rules to follow**: <specific rule or pattern from language assets>
- **Verification command**: `<build, compile, or lint command, or existing test command>`

## Verification rules
- Run the verification command after each step.
- Do not proceed to the next step if verification fails.
- Do not create new unit tests. If existing tests fail due to intentional contract changes, update them to match.
```

---

## Template for closure.md

```markdown
# Task closure: <short title>

## Documentation and assets to update
List what must be created or updated in .agent/assets/ when implementation succeeds.
- **Architecture**: <update system.md or create modules/<name>.md, or "None">
- **Decisions (why)**: <create .agent/assets/why/<component>/<decision>.md, or "None">
- **Language guide**: <add newly introduced project idiom to .agent/assets/language/<lang>.md, or "None">
- **Domain**: <update general.md or create topics/<topic>.md, or "None">

## Handoff checklist
- [ ] All atomic steps in coder-plan.md verified and passed.
- [ ] Diff inspected against scope boundary in intro.md.
- [ ] Required asset updates in .agent/assets/ completed.
- [ ] .agent/scope/ files ready to be archived or cleaned up.
```

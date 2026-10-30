---
name: domain
description: Discover, document, and maintain business domain concepts, workflows, entities, and invariants in .agent/assets/domain/ by grilling the user rather than guessing intent.
disable-model-invocation: true
---

# Domain

Discover, structure, and document business domain concepts, terminology, lifecycles, and business invariants in `.agent/assets/domain/`.

Domain knowledge represents business reality: customer workflows, external vendor contracts, insurance policies, legal requirements, and system boundaries. The model must never invent, assume, or hallucinate business logic. The user bears ultimate responsibility for the domain model. When business rules or entity behaviors are undefined or ambiguous, this skill runs `grill-me` to elicit the necessary facts from the user.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the entity or component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Workflow

1. **Discovery and user interrogation**:
   - Inspect existing codebase interfaces, data structures, and documentation to discover candidate domain concepts.
   - Run the `grill-me` skill workflow for every ambiguous entity, lifecycle transition, or business invariant.
   - Ask questions sequentially. Require the user to define or confirm terms, validation rules, and business relationships.
   - Never extrapolate unproven business logic from code naming alone.

2. **Core domain file (`.agent/assets/domain/general.md`)**:
   - Create or update `.agent/assets/domain/general.md` using the template below.
   - Describe the business purpose, core actors, primary lifecycles, and system boundary with external systems.

3. **Granular topic files (`.agent/assets/domain/topics/<topic>.md`)**:
   - For distinct business workflows, products, or regulatory areas, create dedicated topic files (e.g. `mbr.md`, `service-purchase.md`).
   - Use kebab-case filenames under `.agent/assets/domain/topics/`.
   - Document specific state machines, field constraints, error semantics, and non-negotiable business invariants.

---

## Template for assets/domain/general.md

```markdown
# Business domain overview

## Core purpose
1 to 2 sentences describing the core business service provided and the real-world problem it solves.

## Primary actors and roles
- **<Actor name>**: <Role and permissions within the domain.>
- **<Actor name>**: <Role and permissions within the domain.>

## System boundary and external integrations
- **<Local system>**: <What this application owns and validates.>
- **<External platform>**: <What the upstream or downstream external provider owns and guarantees.>

## Core entity glossary
- **<Term>**: <Precise business definition. Distinguish from technical variable names.>
- **<Term>**: <Precise business definition.>

## Universal business invariants
- Always: <Non-negotiable business rule across all workflows.>
- Never: <Forbidden business action or transition.>
```

---

## Template for assets/domain/topics/<topic>.md

```markdown
# Domain topic: <Topic title>

## Purpose and context
2 to 3 sentences describing this specific business capability, product, or workflow.

## Lifecycle and states
State transitions and trigger events:
- `<State A>` -> `<State B>`: <Trigger event, validation criteria, and actor.>
- `<State B>` -> `<State C>`: <Trigger event, validation criteria, and actor.>

## Field and data constraints
- **<Field name>**: <Format, requirements, and business rules.>
- **<Field name>**: <Format, requirements, and business rules.>

## Business invariants
- Always: <Mandatory behavior or invariant for this topic.>
- Never: <Forbidden behavior or invalid state transition.>
```

---
name: note
description: Record user instructions, hints, constraints, or observations into .agent/scope/notes.md during conversation.
disable-model-invocation: true
---

# Note

Record user notes, constraints, and mid-flight thoughts into `.agent/scope/notes.md`. The note file provides an active scratchpad where user guidance can be captured without altering existing task contracts in `intro.md` or `coder-plan.md`.

## Style and writing rules

All output produced by this skill must follow `.agent/skills/unslop/SKILL.md`:
- State what the system does, not how it feels (Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29). Name the component acting.
- Use plain words over metaphors (Rules 7, 26, 31). Avoid filler words and buzzwords.
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).

## Workflow

1. Check if `.agent/scope/notes.md` exists. If not, create it using the template below.
2. When the user provides a hint, constraint, or design consideration in conversation, classify it:
   - **Constraints and hints**: Non-negotiable requirements, user preferences, or edge-case warnings.
   - **Observations**: Context findings, scratch ideas, or temporary thoughts.
3. Append or update the item under the appropriate section in `.agent/scope/notes.md`.
4. Keep entries concise, concrete, and factual.

---

## Template for notes.md

```markdown
# Task notes

## Constraints and hints
- <User constraint or guidance>

## Observations
- <Context finding or observation>
```

---
name: why
description: Document architectural, design, and structural decisions partitioned by system component in .agent/assets/why/<component>/<decision>.md, deriving evidence from conversation and codebase history or asking the user when unproven.
disable-model-invocation: true
---

# Why

Document why specific technical, architectural, or structural decisions were made across the system. Output is partitioned by component under `.agent/assets/why/<component>/<decision>.md` to give future agents and developers clear context and non-negotiable invariants.

## Evidence hierarchy and asking rule

Never guess or hallucinate historical intent. Gather rationale using this strict evidence order:
1. **Conversation transcript**: Prior discussion, explicit requirements, or agreements made with the user.
2. **Codebase and repository history**: Git commit messages, existing code comments, issue references, or configuration constraints.
3. **Ask the user**: If the rationale cannot be proven from conversation or code, stop and ask the user directly. State your hypothesis and recommended rationale, and wait for confirmation before writing the file.

## Style and writing rules

Follow the direct writing guidelines from `.agent/skills/unslop/SKILL.md`:
- State concrete facts and mechanisms, not feelings (Rule 27).
- Prefer active voice (Rule 29).
- Plain words over metaphors (Rules 7, 26, 31).
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).
- Keep total file length under 30 lines.

## Workflow

1. Identify the relevant system component or module (e.g. `auth`, `mcp-server`, `storage`, `transport`).
2. Verify the rationale using the evidence hierarchy. If unverified, ask the user before proceeding.
3. Choose a concise kebab-case decision slug (e.g. `use-sync-map.md`, `custom-json-rpc-parser.md`).
4. Write or update `.agent/assets/why/<component>/<decision>.md` using the template below.

---

## Template for assets/why/<component>/<decision>.md

```markdown
# Decision: <Short title>

## 1. Decision
1 sentence stating the exact technical or structural choice made.

## 2. Problem
1 to 2 sentences describing the problem being solved. Specify if this is a new feature, a bug fix, a performance bottleneck, or an architectural refactor.

## 3. Why this option and alternatives
1 to 2 sentences explaining why this option was chosen. Name the alternatives considered and the concrete reason each lost (e.g. dependency weight, complexity, lack of standard library support).

## 4. Invariants
Bullet points listing the non-negotiable rules future agents and developers must preserve. State what must not be broken, removed, or refactored away.
- Always: <required behavior or constraint>
- Never: <forbidden pattern or rollback>
```

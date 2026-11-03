---
name: language
description: Create or update targeted language-specific conventions and LLM blind-spot rules in .agent/assets/language/<lang>.md only when needed.
disable-model-invocation: true
---

# Language

Create and maintain high-density, language-specific guidelines focused on idiomatic conventions, built-in standard library defaults, and common LLM blind spots.

## When to run

Run this skill only when needed:
- A new programming language is introduced to the repository.
- A model makes repeated language-specific mistakes that need codifying as permanent guardrails.
- The user explicitly requests a language guide for a stack.

Do not create guides proactively for languages not actively used in the codebase.

## Style and writing rules

Write directly in the target tone from the start:
- State what the code must do, not how it feels (Unslop Rule 27). Name mechanisms and concrete behaviors.
- Prefer active voice (Rule 29).
- Sentence case headings (Rule 17). Straight quotes only (Rule 19). No decorative emojis (Rule 18).
- No em dashes (Rule 13). Use periods or commas.
- One idea per sentence (Rule 28).
- Keep total length under 60 lines. Focus on high-signal guardrails rather than teaching the language from scratch.

## Workflow

1. Identify the target language and inspect existing codebase files to match local project conventions first.
2. Draft or update `.agent/assets/language/<lang>.md` (kebab-case, e.g. `go.md`, `typescript.md`, `python.md`).
3. Follow the strict 3-section template below.

---

## Template for assets/language/<lang>.md

```markdown
# <Language> conventions and LLM guardrails

## Idiomatic conventions
- **Formatting and style**: Canonical tooling (e.g. gofmt, prettier, ruff) and naming patterns (casing, package names, file names).
- **Error handling**: Preferred error wrapping, propagation, and return conventions. Avoid unhandled edge cases or panic equivalents.
- **Structural layout**: How packages, modules, or exports are structured in idiomatic projects.

## Standard library defaults
List canonical built-in modules to use before considering external dependencies:
- `<task>`: Use built-in `<package/module>` instead of third-party libraries (e.g. logging, HTTP, serialization, testing).
- `<task>`: Built-in feature to prefer.

## LLM blind spots and anti-patterns
Targeted guardrails for mistakes models frequently generate in this language:
- **<Trap 1>**: Concrete anti-pattern models generate, and the required idiom to replace it (e.g. resource leaks, unclosed streams, race conditions).
- **<Trap 2>**: Shadowing, unintended copies, or incorrect concurrency usage.
- **<Trap 3>**: Premature abstractions (interfaces with single implementations, unnecessary wrapper classes).
```

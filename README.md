# Agent Suite

A curated repository of portable agent capabilities (skills) and role specifications (agents) for AI agent harnesses (LobeHub, Claude Code, Cursor, Codex).

## Architecture

This repository cleanly separates **agent capabilities** from **agent roles**:

```text
agent-suite/
  ├── skills/           # Reusable tools, procedures, and guardrails
  │   ├── grill-me/     # Relentless design tree interrogation and sparring
  │   ├── unslop/       # Plain-language rules and anti-AI-slop editorial standard
  │   ├── architecture/ # Maps component boundaries and layers
  │   ├── domain/       # Discovers and structures domain entities
  │   ├── language/     # Curates language-specific conventions and idioms
  │   ├── why/          # Architectural decision records (ADRs) and trade-offs
  │   └── index/        # Indexes and synchronizes living assets with code drift
  │
  └── agents/           # Specialized agent personas and system prompt specs
      ├── planner.md    # Discovery, requirements sparring, and task decomposition
      ├── coder.md      # Surgical implementation and compiler verification
      ├── code-verifier.md # Read-only auditor checking git diffs against invariants
      ├── tester.md     # Dedicated test engineer authoring test suites
      └── test-verifier.md # Auditor running mutation checks on test suites
```

## Core Philosophy

- **Zero guessing**: Clarify requirements and settle architectural choices using `grill-me` before touching code.
- **Living assets**: Maintain persistent architecture, language idioms, and decisions inside target repositories (`.agent/assets/`).
- **Minimal ceremony**: Eliminate ephemeral markdown state machines and procedural overhead. Let the harness manage execution and let the agents focus on the work.
- **Role separation**: Feature coders do not self-audit diffs or self-author tests; dedicated auditors verify diffs and test suites.

## Attribution

- `grill-me` is based on [Matt Pocock's grilling skill](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling).
- `unslop` is based on [Cursor's unslop skill](https://github.com/cursor/plugins/blob/1ba91fc25c0977741dd0e267849e6234b037ed0a/pstack/skills/unslop/SKILL.md).

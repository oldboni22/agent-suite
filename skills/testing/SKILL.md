---
name: testing
description: Discover, document, and maintain repository testing conventions, runners, assertions, and mutation paths in .agent/assets/testing/conventions.md.
disable-model-invocation: true
---

# Testing

Discover, document, and maintain repository-specific test conventions, runners, assertion libraries, and mutation testing paths in `.agent/assets/testing/conventions.md`.

## Style and writing rules

All output produced by this skill must comply with `unslop` and `honest` rules.

## Workflow

1. Inspect repository package manifests, build scripts, and existing test directories to identify:
   - Primary test runner and framework (e.g. `go test`, `pytest`, `vitest`, `cargo test`).
   - Assertion library and mocking pattern (e.g. standard library, testify, jest mocks).
   - Test placement (colocated `_test.go` / `*.spec.ts` vs. top-level `tests/`).
   - Mutation testing tools or native red-green perturbation commands.
2. If test conventions are ambiguous or multiple runners exist, grill the user using `grill-me` to confirm canonical commands.
3. Write or update `.agent/assets/testing/conventions.md` using the template below.

---

## Template for assets/testing/conventions.md

```markdown
# Testing conventions

## Canonical runners and commands
- **Unit tests**: `<canonical command to run all unit tests, e.g. go test -v ./... or npm test>`
- **Single test file**: `<command pattern to run a single test file>`
- **Targeted single test**: `<command pattern to run a specific test by name/pattern>`
- **Coverage check**: `<command to generate test coverage report>`

## Test layout and structure
- **File location**: `<colocated with source | separate test directory>`
- **Naming convention**: `<e.g. *_test.go, *.test.ts, test_*.py>`
- **Test style**: `<e.g. table-driven tests, BDD describe/it blocks, class-based>`

## Mocking and fixtures
- **Mocking library**: `<preferred mock library or in-house interfaces/fakes>`
- **Fixtures and test data**: `<location and format for test fixtures>`
- **External boundaries**: `<rules for mocking network, filesystem, database, or time>`

## Mutation and quality gates
- **Mutation runner**: `<native mutation tool command if available, or "synthetic perturbation">`
- **Quality invariants**:
  - Tests must fail if production logic is perturbed (non-vacuous).
  - Feature coders never author or modify test files.
  - Dedicated testers never modify production code to make tests pass.
```

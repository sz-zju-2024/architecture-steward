# Harness Invocation

A harness is a repeatable quality gate. It can be manual at first, then scripted, then integrated into CI or exposed through MCP tools.

## Common Invocation Styles

### Manual prompt invocation

Use when the project is new or the checks are still evolving:

```text
Use $architecture-steward to review my latest changes. Check layer boundaries, reusable abstractions, public/internal exports, and missing tests.
```

### Local command

Use when checks are stable enough to script:

```bash
npm run arch:check
pnpm arch:check
yarn arch:check
make arch-check
python scripts/arch_check.py
```

The command should print actionable findings and exit non-zero only for clear violations.

### Git hook

Use for cheap checks only:

```text
pre-commit: formatting, lint, import boundary rules
pre-push: typecheck, targeted tests, architecture check
```

Avoid slow e2e tests in pre-commit.

### CI gate

Use for team consistency:

```text
pull_request:
  - install
  - lint
  - typecheck
  - unit tests
  - architecture/import boundary checks
  - targeted integration/e2e tests
```

### AI workflow gate

Use after an AI coding pass:

```text
1. inspect changed files
2. classify each file by layer
3. check imports against allowed directions
4. detect new public exports
5. identify missing or misplaced tests
6. summarize risks and recommended fixes
```

## Minimal First Harness

Start with a simple script or checklist that answers:

- Which files changed?
- What layer does each changed file belong to?
- Do imports cross forbidden boundaries?
- Are system APIs only used in infrastructure?
- Did UI call services instead of persistence/native APIs?
- Did shared utilities import app-specific code?
- Are tests near the changed logic?
- Are schema or persistence changes paired with migrations?
- Are desktop process/API boundaries crossed only through explicit contracts?

If the project is still evolving, run this as an agent checklist first. Script only the rules that are stable, mechanical, and low-noise.

## What To Script First

Good first checks:

- Forbidden imports between layers.
- Renderer/UI imports of filesystem, SQLite, Electron main modules, or native APIs.
- Domain/shared imports of React, DOM, browser storage, or feature modules.
- Schema/model file changes without migration file changes.
- New public exports from package entrypoints.
- Missing nearby test files for changed domain/service/persistence code.

Keep these as agent judgment rather than scripts until patterns stabilize:

- Whether an abstraction is premature.
- Whether a service boundary is expressive enough.
- Whether a refactor should be done now or tracked as debt.
- Whether a feature should become a package.

## Suggested Output Format

```text
Architecture check

Changed layers:
- renderer: ...
- services: ...
- infra: ...

Findings:
- [High] renderer imports filesystem adapter directly at ...
- [Medium] new shared utility has only one caller at ...
- [Medium] public export may be internal at ...
- [High] model changes are not paired with migration handling at ...

Suggested fixes:
- route save flow through noteService
- move filesystem call to infra/noteRepository
- keep helper local until a second caller appears
- add a versioned migration and a migration test/manual verification note
```

## When To Add MCP

Add MCP only after the harness has stable questions that need faster codebase lookup:

- find symbol definition
- find callers/importers
- map module dependency graph
- locate related tests
- explain a feature's ownership boundaries

Do not build MCP first if the real need is still deciding what the architecture rules should be.

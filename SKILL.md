---
name: architecture-steward
description: Architecture stewardship workflow for planning, implementing, and reviewing changes in complex software projects. Use when Codex is asked to design or modify non-trivial code, create or reorganize project structure, enforce layer boundaries, decide whether code should become reusable utilities/packages/services, review AI-generated code quality, or build a harness/checklist that keeps UI, system APIs, business logic, persistence, tests, and public/internal APIs from becoming tangled.
---

# Architecture Steward

## Overview

Use this skill to preserve architecture quality while changing a complex codebase. Prefer the existing project's conventions, map generic layers onto the repository's actual directories, then apply the boundary and review workflow in this skill.

Grafana-inspired principles and concrete checklists live in `references/grafana-architecture-principles.md`. Harness invocation patterns live in `references/harness-invocation.md`. Report format and severity rules live in `references/review-output.md`.

Before producing an architecture review, MUST read `references/review-output.md` and use its severity levels, required finding shape, validation reporting, and self-evaluation format.

Read these optional references only when the task needs them:

- `references/contracts.md`: when reviewing renderer/main, client/server, plugin host/plugin, mobile/native bridge, public SDK, RPC, message bus, or other API contract boundaries.
- `references/persistence-migrations.md`: when reviewing database, document schema, event schema, file format, cache format, persistence, migration, backfill, or compatibility changes.

## Use Stages

Use this skill at these stages:

1. **Project discovery:** Before a meaningful change, inspect directories, package manifests, build scripts, tests, style guides, and architecture docs.
2. **Feature design:** Before writing code, decide which layer owns UI, domain rules, services, system APIs, persistence, and shared utilities.
3. **Implementation:** While editing, preserve dependency direction and use public entrypoints rather than importing internal files casually.
4. **Pre-review:** After editing, run the architecture checklist before the final answer or pull request summary.
5. **Refactor planning:** When a feature is becoming tangled, propose small boundary-improving steps instead of a broad rewrite.
6. **Harness design:** When asked to automate quality checks, turn these rules into scripts, lint constraints, CI steps, or MCP/query tools.

Do not overuse this skill for tiny one-file changes unless the change touches shared modules, persistence, system APIs, public exports, or test strategy.

## Workflow

### 1. Survey the Project

Read enough local context to understand the current architecture:

- Root manifests: `package.json`, `go.mod`, `pyproject.toml`, `Cargo.toml`, `Makefile`, `nx.json`, `project.json`, CI files.
- Architecture docs: `docs/`, `contribute/`, `CONTRIBUTING.md`, `README.md`, `ARCHITECTURE.md`, `adr/`.
- Source layout: UI/app directories, feature directories, service/domain directories, infrastructure/persistence directories, shared packages, tests.
- Existing examples near the change target.

Summarize the detected layers by mapping the repository's real directories to the generic model before making broad design choices. If the mapping is uncertain, state the uncertainty instead of inventing a new architecture.

Also identify validation commands from local project metadata. Prefer existing scripts such as `typecheck`, `build`, `test`, `lint`, package-specific test commands, or Makefile targets. Do not hardcode commands from another project.

### 2. Assign Ownership

Before editing, classify the requested behavior:

- UI presentation and interaction belongs in UI/renderer/components/features.
- Business rules and invariants belong in domain or service/use-case modules.
- Workflow orchestration belongs in services, commands, controllers, or application layer.
- Filesystem, database, HTTP clients, OS APIs, Electron/Tauri/native APIs belong in infrastructure adapters.
- Stable reusable primitives belong in shared packages or public entrypoints.
- Experimental or app-private code belongs behind internal or unstable entrypoints.
- Renderer/main/preload or client/server boundaries must communicate through explicit API contracts, DTOs, or typed channels rather than shared implementation imports.
- Database schema changes belong in migrations or versioned schema upgrade paths, not incidental startup code.
- Contract files may define API shapes, DTOs, channel names, schemas, error shapes, and serialization rules. They must not import UI components, persistence clients, native/system APIs, concrete service implementations, or view state.

If a change seems to require crossing layers, introduce a small interface, adapter, repository, or service method rather than importing lower-level implementation details into UI code.

### 3. Preserve Dependency Direction

Prefer this direction:

```text
UI/renderer -> services/application -> domain -> shared
services/application -> infra adapters
infra adapters -> shared
tests -> public or test-only helpers
```

Avoid this direction:

```text
UI/renderer -> database/filesystem/native APIs directly
domain -> React/Electron/browser/localStorage
shared/utils -> feature-specific modules
feature A -> feature B internal files
infra -> UI
public package -> app-private implementation
renderer -> main/preload implementation files directly
main/preload -> renderer UI components or view state
schema/model changes -> persistence code without migration handling
```

When a repository has its own architecture rules, follow those first and use this skill as a gap-filling checklist.

### 4. Decide Reuse Conservatively

Extract a reusable function, component, service, or package only when most of these are true:

- There are at least two real callers, or the code is clearly a stable project boundary.
- Inputs and outputs are explicit and testable.
- The extraction reduces dependencies instead of requiring many injected details.
- The new abstraction has a name based on domain meaning, not current implementation trivia.
- It does not expose private implementation details as a public API.
- It can be tested without UI or external system setup, unless it is explicitly an adapter.

Keep one-off feature logic local. Duplication is often cheaper than a premature shared abstraction.

### 5. Decide Refactor vs. Debt

Require immediate refactoring when:

- The change introduces a forbidden dependency across UI, domain, infrastructure, persistence, or system boundaries.
- The change makes data loss, security exposure, broken migrations, or incompatible public API changes likely.
- New code duplicates a risky workflow in multiple places, such as persistence, permissions, transaction handling, or IPC validation.
- Tests cannot reasonably cover the behavior because the code is tangled with UI/system/persistence concerns.

Record technical debt instead of refactoring immediately when:

- The issue is pre-existing and outside the current change path.
- The fix would require broad repo restructuring for a small feature.
- The duplication is local, low-risk, and not yet a stable abstraction.
- The project lacks enough examples to define the right abstraction confidently.

When recording debt, name the owner area, risk, suggested future trigger, and smallest next step.

### 6. Review After Editing

Before finalizing, perform an architecture review:

- Did new code land in the layer that owns the behavior?
- Did UI avoid direct persistence, filesystem, OS, network, or native API access?
- Did domain/business logic avoid React, DOM, Electron/Tauri, browser storage, and database dependencies?
- Did infrastructure remain mostly adapters/clients/repositories rather than business policy?
- Did shared utilities remain generic and dependency-light?
- Did public exports stay intentional? Should the new export be public, unstable, internal, or private?
- For desktop apps, are renderer/main/preload contracts explicit and typed, with implementation details kept on the owning side?
- For persistence changes, are schema changes paired with migrations, compatibility handling, and tests or manual verification?
- Are tests added at the right level: pure unit tests for domain/utils, service tests for orchestration, adapter tests for infrastructure, component/e2e tests for UI workflows?
- Are existing project commands used for validation?

Report risks and test gaps honestly. Use severity levels and the output format in `references/review-output.md`. If validation cannot run because dependencies, tools, credentials, services, network, or generated artifacts are missing, report what was attempted, why it failed, how that limits confidence, and the smallest recovery step.

## Harness Pattern

When asked to build or use a harness, make it a small repeatable gate first:

```text
discover -> classify changed files -> run cheap checks -> run targeted tests -> architecture checklist -> report
```

Start with scripts that read project metadata and changed files. Add deeper symbol graphs or MCP tools later only when the checklist reveals recurring blind spots.

Read `references/harness-invocation.md` when the user asks how to automate the checklist.

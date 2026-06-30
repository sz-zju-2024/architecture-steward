# Grafana-Derived Architecture Principles

Use these principles as inspiration, not as rules to copy blindly.

## What Grafana Teaches

Grafana's repository separates code by responsibility and API stability:

- `public/app/`: main frontend application.
- `public/app/features/`: feature-oriented frontend modules.
- `public/app/core/`: frontend platform/core facilities.
- `public/app/plugins/`: built-in plugin implementations.
- `packages/`: reusable frontend packages such as UI, data, runtime, schemas, plugin configs, and test utilities.
- `pkg/`: Go backend application code.
- `pkg/services/`: backend services that encapsulate application logic.
- `pkg/api/`: HTTP API layer.
- `pkg/server/`: service lifecycle and dependency wiring.
- `apps/`: newer app/API modules and resource-style backend components.
- `contribute/`: architecture, backend, frontend, testing, and style guides.

The transferable idea is not the exact folder names. The transferable idea is to separate code by ownership, dependency direction, and stability.

## Public, Unstable, Internal

Grafana packages distinguish stable public exports from experimental and internal exports. Use the same idea in other projects:

- `public`: stable API intended for multiple modules or plugins.
- `unstable`: experimental API that can change.
- `internal`: shared inside the app but not for external consumers.
- private files: implementation details for one feature/module.

Before exporting a function or type, ask:

- Who is the intended caller?
- Is the API stable enough to support?
- Can callers use a narrower service method instead?
- Would exporting this expose implementation details?

## Desktop App Mapping

For Electron, Tauri, or similar desktop apps, prefer a structure like:

```text
src/
  renderer/       UI, pages, feature components, view state
  domain/         pure business models, validation, invariants
  services/       application use cases and workflow orchestration
  infra/          filesystem, database, OS APIs, IPC, network clients
  shared/         dependency-light types, constants, pure utilities
  tests/          test helpers and integration/e2e tests
  docs/           architecture notes and ADRs
```

Allowed dependency direction:

```text
renderer -> services
services -> domain
services -> infra
domain -> shared
infra -> shared
```

Forbidden or suspicious dependency direction:

```text
renderer -> fs/sqlite/electron/native APIs directly
domain -> renderer/react/dom/browser/electron
shared -> renderer/features/services/infra
infra -> renderer
feature -> another feature's internal files
```

## Desktop Boundary Contracts

For Electron, Tauri, mobile, or client/server apps, treat process and privilege boundaries as architecture boundaries:

- Renderer/UI code owns presentation, user interaction, view state, and calls to a narrow app API.
- Main/native/system code owns filesystem, database, OS integration, native windows, background jobs, and privileged APIs.
- Preload/bridge code exposes a small typed contract, not broad access to implementation modules.
- Shared contract files may contain DTOs, request/response types, channel names, validation schemas, and stable API shapes.
- Shared contract files should not import UI components, database clients, native modules, or implementation services.

Prefer this:

```text
renderer -> typed appApi contract -> preload bridge -> main service -> repository/adapter
```

Avoid this:

```text
renderer -> electron main implementation
renderer -> sqlite/filesystem/native module
preload -> imports feature UI state
shared contract -> imports implementation classes
```

Review contract changes like public API changes. Ask whether the contract is stable, narrow, serializable, and validated at the boundary.

## Persistence and Migration Rules

Treat persistence as a high-risk boundary:

- Schema changes require a migration, versioned upgrade path, or explicit compatibility handling.
- Startup code may run migrations, but should not hide ad hoc schema creation or destructive changes without review.
- Repository/DAO code should not own business policy that belongs in services/domain.
- UI code should not build SQL, call SQLite directly, or make assumptions about storage layout.
- Migrations should be deterministic, ordered, idempotent when appropriate, and tested or manually verifiable.
- Data shape changes should include backfill/default handling and rollback or recovery notes when risk is meaningful.

When reviewing persistence changes, look for:

- New or changed tables/columns/indexes.
- Raw SQL added outside migration/repository layers.
- Model fields changed without migration.
- Serialization format changes.
- Deletion or overwrite behavior.
- Missing transaction boundaries for multi-step writes.

## Boundary Examples

Saving a note should look like:

```text
SaveButton -> noteService.saveNote -> validateNote -> noteRepository.save -> filesystem/database
```

It should not look like:

```text
SaveButton -> validate title + write file + update database + notify OS + mutate config
```

## Reuse Decisions

Prefer local code when:

- There is only one caller.
- The behavior is feature-specific.
- The name would include page or workflow details.
- Extraction would require many parameters or hidden globals.

Prefer shared utilities when:

- The function is pure and dependency-light.
- Two or more modules need the same behavior.
- The name describes a general concept.
- It can be tested with simple inputs and outputs.

Prefer services when:

- The code coordinates multiple operations.
- The UI should not know persistence/system details.
- The behavior is a user-visible use case.

Prefer infrastructure adapters when:

- The code touches files, database, HTTP, OS, browser storage, Electron/Tauri IPC, or external services.
- Tests need to mock or fake the external boundary.

## Review Questions

Use these questions before finalizing a complex change:

- What layer owns this behavior?
- Does the import graph match that ownership?
- Is any system API leaking into UI or domain code?
- Is business policy hiding inside adapters or components?
- Are public exports intentional?
- Are tests placed at the same layer as the logic being tested?
- Is the smallest useful abstraction enough?

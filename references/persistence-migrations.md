# Persistence and Migration Patterns

Use this reference when reviewing persistent data shape changes. It is intentionally stack-neutral: choose the pattern that matches the repository instead of copying a specific tool's example.

## What Counts As Persistence

- SQL tables, columns, indexes, constraints, triggers, and views.
- Document schemas and embedded JSON payloads.
- Event schemas and message formats.
- File formats and export/import formats.
- Cache formats when old cache data can affect correctness.
- Local storage, browser storage, mobile storage, and desktop app user data.

## General Migration Requirements

Persistence changes should have:

- A versioned or ordered upgrade path.
- Deterministic and repeatable migration steps.
- Backfill or default handling for existing data.
- Compatibility notes when old app versions may read new data or new app versions may read old data.
- Recovery or rollback notes when data loss risk is meaningful.
- Tests or manual verification instructions appropriate to the risk.

## Common Patterns

### Ordered Migration Files

Used by many server frameworks and database tools.

```text
migrations/
  001_initial_schema.sql
  002_add_entry_references.sql
```

Review for ordering, idempotency expectations, transaction behavior, and backfill.

### Schema Version Field

Used by embedded databases, local app stores, document stores, and file formats.

```text
currentVersion = readStoredVersion()
if currentVersion < 1: migrateToV1()
if currentVersion < 2: migrateToV2()
writeStoredVersion(2)
```

For SQLite, one implementation option is `PRAGMA user_version`, but this is only one example. Do not require it for non-SQLite projects.

### ORM or Framework Migrations

Used by tools such as Prisma, Django, Rails, TypeORM, EF, or Alembic.

Review the generated migration plus the application model change. Do not assume the model change alone is enough.

### Document or Event Schema Versions

Used when data is serialized into JSON, events, messages, or files.

```json
{
  "schemaVersion": 2,
  "payload": {}
}
```

Review readers, writers, compatibility, and replay/import behavior.

## Review Questions

- What existing data will be present before this change?
- How does old data become readable by the new code?
- Can the migration run more than once? If not, how is ordering guaranteed?
- Are destructive operations explicit and justified?
- Is multi-step migration wrapped in a transaction or recovery-safe process?
- Are indexes/constraints/triggers included when query behavior depends on them?
- Is the app protected from opening a database/file with a newer unsupported version?
- How was migration verified?

## Severity Hints

- Critical: likely data loss, destructive migration without recovery, startup failure for existing users, or unsupported public data format break.
- High: schema/model change without migration or compatibility handling.
- Medium: migration exists but lacks backfill, verification, or ownership clarity.
- Low: migration naming or documentation could be clearer.

## Minimal Manual Verification

When tests do not exist yet, report at least:

- Fresh install path: empty store migrates to latest schema.
- Upgrade path: old store migrates to latest schema.
- Idempotency or ordering behavior.
- A representative read/write after migration.

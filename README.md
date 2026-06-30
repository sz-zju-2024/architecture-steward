# Architecture Steward

Architecture stewardship workflow for planning, implementing, and reviewing changes in complex software projects.

This Codex skill helps keep UI, system APIs, business logic, persistence, tests, and public/internal APIs from becoming tangled while a codebase evolves.

## What It Helps With

- Mapping an existing repository's real directories to architectural layers.
- Deciding where new behavior belongs before implementation.
- Preserving dependency direction across UI, services, domain, infrastructure, and shared code.
- Reviewing AI-generated or human-written changes for architectural risk.
- Checking contract boundaries such as renderer/main, client/server, plugin host/plugin, SDK, RPC, or message bus APIs.
- Reviewing persistence, schema, migration, compatibility, and backfill changes.
- Designing repeatable architecture review harnesses for CI or local checks.

## When To Use It

Use this skill when working on non-trivial code changes, project structure changes, reusable packages, public APIs, persistence changes, or architecture reviews.

It is intentionally lightweight for small edits: do not use it for tiny one-file changes unless they touch shared modules, persistence, system APIs, public exports, or test strategy.

## Repository Structure

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── contracts.md
    ├── grafana-architecture-principles.md
    ├── harness-invocation.md
    ├── persistence-migrations.md
    └── review-output.md
```

## Installation

Clone this repository into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/sz-zju-2024/architecture-steward.git ~/.codex/skills/architecture-steward
```

Then restart or reload your Codex session so the skill can be discovered.

## Usage

Ask Codex to use the `architecture-steward` skill when you want architectural guidance or review, for example:

```text
Use the architecture-steward skill to review this change before I merge it.
```

```text
Use architecture-steward to help design this feature without crossing UI, service, and persistence boundaries.
```

```text
Build a small architecture review harness for this repository using architecture-steward.
```

## Key References

- `references/review-output.md`: required architecture review output format and severity rules.
- `references/contracts.md`: contract boundary checks for APIs, channels, DTOs, and serialization.
- `references/persistence-migrations.md`: persistence, schema, migration, and compatibility checks.
- `references/harness-invocation.md`: patterns for repeatable review harnesses.
- `references/grafana-architecture-principles.md`: architecture principles and practical checklists.

## License

No license has been selected yet. Add one before reusing or redistributing this skill outside personal or internal workflows.

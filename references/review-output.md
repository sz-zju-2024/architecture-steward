# Review Output

Use this format for architecture review reports. Keep findings evidence-based and prioritized.

## Severity Levels

- **Critical:** Likely data loss, security issue, migration breakage, public API breakage, or application startup/runtime failure.
- **High:** Clear boundary violation, hard-to-test coupling, direct system/persistence access from UI/domain, unsafe IPC/native API exposure, or missing migration for schema change.
- **Medium:** Maintainability risk, unclear ownership, over-broad public export, repeated workflow logic, misplaced tests, or abstraction likely to spread.
- **Low:** Naming, organization, documentation, small test gap, or local duplication that does not currently block safe change.
- **Info:** Context, assumptions, or improvement ideas that are not problems yet.

Do not assign high severity to stylistic preferences. Do not assign low severity to data/schema/security risks.

## Required Finding Shape

Every finding must include:

- Severity.
- File path and line number when available.
- Evidence: import, function, component, command, schema change, or observed pattern.
- Why it matters.
- Smallest useful fix.

Avoid findings that only say "consider improving architecture" without concrete evidence.

## Report Template

```text
Architecture Review

Scope:
- Files/areas inspected:
- Project layer mapping:
- Validation commands discovered:

Findings:
- [High] UI layer directly calls persistence
  Evidence: src/... imports ...
  Why it matters: ...
  Smallest fix: ...

- [Medium] Shared utility may be premature
  Evidence: ...
  Why it matters: ...
  Smallest fix: keep local until ...

Validation:
- Ran:
- Not run:
- Failed:
- Confidence impact:
- Recommended next commands:

Refactor vs Debt:
- Must refactor now:
- Safe to track as debt:

Skill Feedback:
- Missing rule:
- Ambiguous rule:
- False-positive risk:
- Self-evaluation:
```

## Validation Command Guidance

Discover commands from local metadata. Common names include:

- JavaScript/TypeScript: `npm run typecheck`, `npm run build`, `npm test`, `npm run lint`, or equivalent `pnpm`/`yarn` scripts.
- Go: `go test ./...`, `make test`, `make lint`.
- Rust: `cargo test`, `cargo clippy`.
- Python: `pytest`, `ruff`, `mypy`.

Use the project's actual package manager and scripts. If no command exists, state that instead of inventing one.

## Validation Failure Reporting

When validation cannot run or does not complete, report it as evidence rather than hiding it.

Include:

- Command attempted.
- Observed failure, using the concrete error when useful.
- Likely category: missing dependencies, missing toolchain, unsupported runtime, missing credentials, unavailable service, network failure, generated files missing, or unknown.
- Confidence impact: what conclusions are still supported by inspection, and what remains unverified.
- Smallest recovery step: install dependencies, switch runtime, start service, provide credentials, run generation, or rerun a narrower command.

Do not claim a check passed when only a weaker substitute ran. For example, dependency installation with lifecycle scripts disabled can support static type checks, but it does not prove native modules or runtime startup work.

## Self-Evaluation

When using this skill to test itself or when the user asks for skill quality feedback, use this fixed shape:

```text
Skill Self-Evaluation

Score: x/10

Strengths:
- ...

Ambiguities:
- ...

False-positive risks:
- ...

Missing decision rules:
- ...

Over-specificity risks:
- ...

Suggested next revision:
- ...
```

Rate the result by:

- Did it map actual project directories before applying generic rules?
- Did it provide concrete evidence for each finding?
- Did severity reflect risk, not taste?
- Did it separate required refactors from technical debt?
- Did it discover validation commands from the project?
- Did it avoid broad rewrites unless repeated high-risk violations were found?
- Did it stay general enough for different stacks while still giving actionable guidance?

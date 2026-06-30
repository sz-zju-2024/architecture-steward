# Contract Boundary Guidance

Use this reference when reviewing boundaries between independently owned layers or processes, such as renderer/main/preload, client/server, plugin host/plugin, mobile/native bridge, public SDKs, RPC, message buses, workers, or service APIs.

## Purpose

A contract is a stable boundary shape. It should let one side call or exchange data with another side without importing the other side's implementation.

Good contracts make changes safer by separating:

- What callers may rely on.
- What implementations may change privately.
- What data crosses a trust, process, package, runtime, or deployment boundary.

## Contract Files May Contain

- DTOs and serializable request/response types.
- Channel, route, procedure, event, or message names.
- Public API interfaces.
- Error shapes and status codes.
- Validation schemas or schema references.
- Capability flags and version markers.
- Serialization rules and compatibility notes.

## Contract Files Must Not Contain

- UI components, hooks, view models, or view state.
- Database clients, filesystem clients, network clients, native modules, or system APIs.
- Concrete service implementations.
- Repository/DAO implementations.
- Runtime wiring, dependency injection containers, app startup code, or framework-specific state.
- Test-only helpers unless the contract is explicitly a test contract.

## Review Questions

- Is the contract narrow enough for the current use case?
- Is the contract serializable across the boundary?
- Does it expose implementation details or private storage shape?
- Are errors and unsupported capabilities represented explicitly?
- Is the contract versioned or compatible enough for the deployment model?
- Are validation responsibilities clear on at least one side of the boundary?
- Are imports pointing toward the contract rather than toward implementation files?

## Common Findings

- High: caller imports server/main/native implementation directly instead of using the contract.
- High: contract exposes privileged filesystem, database, or native capability too broadly.
- Medium: contract contains implementation-specific types that force callers to know storage internals.
- Medium: contract grows into a general service locator instead of a narrow capability.
- Low: contract naming is unclear but the boundary is otherwise safe.

## Smallest Useful Fixes

- Move DTOs and API shape into a shared contract file or package.
- Replace implementation imports with contract imports.
- Add an adapter on each side of the boundary.
- Split one broad contract into smaller capability groups.
- Add explicit error/status/capability fields instead of relying on thrown implementation errors.

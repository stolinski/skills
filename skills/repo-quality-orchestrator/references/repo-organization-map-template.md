# Repository Organization Map

This file is the placement contract for source files.
Update it when folder ownership or dependency boundaries change.

## Global principles

- Prefer small focused files and cohesive modules.
- Keep dependency direction one-way across layers.
- Avoid circular dependencies across folders.
- Put shared utilities in explicitly shared locations only.

## Folder contracts

### `src/features/*`

- Purpose: feature-local UI/domain/application code.
- Include: components, services, feature-scoped helpers.
- Exclude: global infrastructure, generic cross-feature utilities.
- Allowed imports: `src/shared/*`, `src/lib/*`, same feature.
- Disallowed imports: other feature internals (except approved public entrypoints).
- Ownership notes: feature owners maintain this folder.

### `src/shared/*`

- Purpose: reusable primitives and cross-feature modules.
- Include: shared UI primitives, shared pure utilities, shared domain types.
- Exclude: feature-specific business logic.
- Allowed imports: `src/lib/*` only.
- Disallowed imports: `src/features/*` internals.
- Ownership notes: changes require broad compatibility checks.

### `src/lib/*`

- Purpose: infrastructure and external integration boundaries.
- Include: API clients, persistence adapters, framework setup.
- Exclude: feature presentation logic.
- Allowed imports: low-level runtime packages and config.
- Disallowed imports: `src/features/*` and `src/shared/*` UI code.
- Ownership notes: enforce strict public APIs for adapters.

## Placement review checklist

- Does this file match the folder purpose?
- Is it importing only allowed layers?
- Is related code grouped in the same module area?
- Could this module be moved to a narrower scope?

## Move policy

- Prefer local moves over broad tree rewrites.
- Preserve public import paths unless explicitly approved.
- Batch moves in small groups, then run full verification.

## Change log

- YYYY-MM-DD: Initial map created.

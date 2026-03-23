---
name: repo-quality-orchestrator
description: Orchestrates sub-agents to audit and safely clean up messy TypeScript codebases. Use when asked to "clean up AI-generated code", "reduce duplication", "improve folder organization", "enforce modularity", or "grade code quality".
metadata:
  version: 1.0.0
---

# Repo Quality Orchestrator

Use this skill to grade and remediate TypeScript code quality without changing intended behavior.

## Primary objective

Clean up messy, repetitive AI-generated code while preserving behavior, public contracts, and delivery safety.

## Required files

- `plan/code-quality-audit-tracking.md`: append-only audit log for every analyzed/updated file.
- `plan/repo-organization-map.md`: source of truth for folder intent and dependency boundaries.

If either file is missing, create it from:

- `references/code-quality-audit-tracking-template.md`
- `references/repo-organization-map-template.md`

## Rubric (1-5 per dimension)

| Dimension | Weight | What it measures |
|---|---|---|
| Structural organization and boundaries | High | Clear local architecture and low cross-layer leakage |
| File placement and folder cohesion | High | Correct folder placement and related code grouped together |
| Encapsulation and API surface | High | Narrow exports, internal details hidden, clear contracts |
| Modularity and cohesion | High | Small focused files and single-purpose functions |
| DRY and abstraction quality | High | Removes duplication without over-abstraction |
| Testability and determinism | High | Code is easy to test and avoids hidden side effects |
| Type-level safety | High | Strong TypeScript contracts and safe narrowing |
| Error handling and defensive coding | High | Predictable failures and guarded boundaries |
| Documentation and intent clarity | Medium | Module intent docs are present, concise, and current |
| Regression and change safety | High | Changes are likely to catch future bugs, not create them |

## Scoring method

- Weight mapping: `High=3`, `Medium=2`, `Low=1`.
- Formula: `weighted_percent = sum(score * weight) / (5 * sum(weights)) * 100`.

| Total % | Rating | Priority |
|---|---|---|
| 87-100% | Excellent | No action needed |
| 70-86% | Good | Low priority improvements |
| 50-69% | Adequate | Medium priority - schedule in current phase |
| 30-49% | Weak | High priority - targeted rewrite |
| < 30% | Insufficient | Critical - treat as unsafe/untrusted |

## Oversized file policy (hard gate)

Treat file size as a quality risk indicator, not a standalone verdict.

Tiered LOC thresholds:

- `>250 LOC`: warning - review for split opportunities
- `>350 LOC`: high priority - refactor planning required
- `>500 LOC`: critical - treat as unsafe unless explicitly exempt

Structural trigger checks (apply regardless of LOC):

- More than 10 top-level exports
- Any function longer than 60 lines
- Mixed concerns in one file (for example domain logic + transport + persistence)
- More than 3 side-effectful dependencies in a single module

Scoring caps when oversized:

- Warning tier: cap `Modularity and cohesion` at `3/5`
- High-priority tier: cap `Modularity and cohesion` and `Structural organization and boundaries` at `2/5`
- Critical tier: cap both dimensions at `1/5` unless exempted with explicit rationale

Required remediation flow for oversized files:

1. Create a split plan first (target modules, ownership, API preservation strategy).
2. Refactor in phases (max 2 extractions per pass).
3. Re-run verification after each phase: `lint:fix`, `typecheck`, `build`, `test`, `test:e2e`.
4. If public API break risk is unclear, defer and log an open question instead of guessing.

## Workflow

1. Create or update tracking files and define target batch scope.
2. Process files in small batches (10-20 files max per run).
3. For each file:
   - Score every rubric dimension.
   - Record findings with `severity`, `confidence`, and `evidence`.
   - Record folder placement status against `plan/repo-organization-map.md`.
   - List prioritized improvements.
4. If improvements are high-confidence and safe, implement them immediately.
5. If uncertainty remains, log open questions before editing.
6. Verify with: `lint:fix`, `typecheck`, `build`, `test`, `test:e2e`.
7. Record results, score deltas, unresolved questions, and next batch.

## Evidence format (required)

Every finding must include:

- Severity: `high|medium|low`
- Confidence: `high|medium|low`
- Evidence: `path:line` plus concrete behavior or structure mismatch

Do not include ungrounded opinions or style-only findings.

## Repository organization contract

Use `plan/repo-organization-map.md` to evaluate placement and dependencies.

For each folder in that map, define:

- Purpose: what belongs there
- Exclusions: what does not belong there
- Allowed imports: permitted dependency directions
- Ownership notes: who or what maintains this area

For file moves:

- Move only with high confidence and explicit map evidence.
- Prefer small local moves over broad tree rewrites.
- Preserve public import/API paths unless explicitly approved.
- Batch move limit: max 10 files before re-running verification.

## Module intent docs policy

Every touched TypeScript module should include and maintain this header:

```ts
/**
 * Module: <path-or-name>
 * Intent: <why this module exists>
 * Responsibilities: <what this module owns>
 * Public API: <exports and contracts>
 * Invariants: <rules that must stay true>
 * Side Effects: <none|list>
 * Maintenance: Update this block when exports, invariants, side effects, or ownership change.
 */
```

Keep the block concise and factual.

## Safety boundaries

- Primary objective is cleanup quality, not architecture churn.
- Do not change public APIs/contracts unless explicitly required and documented.
- Do not do broad rewrites in a single pass.
- Do not do speculative refactors without evidence.
- Do not do style-only churn.

Default edit budget per pass:

- Max 3 non-test files touched
- Max ~200 LOC net change

If budget is exceeded, stop and log phased follow-up work.

## Failure policy

If improvements cause failures:

1. Classify the failure:
   - genuine implementation bug
   - ambiguity/design question
2. If genuine bug, fix implementation.
3. If ambiguity, revert risky change and log open question in tracking file.
4. Re-run impacted tests twice to guard against flakes.

## Context-efficiency rules

- Read only target file plus directly related neighbors.
- Avoid full-repo deep reads in one pass.
- Keep reports compact; do not dump full file contents.
- Prefer phased improvements over large all-at-once rewrites.

## Required output format

Return:

1. `Files Audited`
2. `Files Updated`
3. `Score Deltas`
4. `Open Questions`
5. `Verification Results`
6. `Next Batch`

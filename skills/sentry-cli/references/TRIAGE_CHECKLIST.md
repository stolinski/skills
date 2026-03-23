# Sentry CLI Triage Checklist

Use this checklist for a complete issue investigation pass.

## 1) Access and Scope

- The CLI is installed and callable (`sentry --version` or `npx sentry@latest --help`).
- `sentry auth status` succeeds.
- `sentry auth whoami --json` or `sentry whoami --json` is used when identity matters.
- Organization and project are identified via auto-detect first (`sentry project view --json` or `sentry issue list --json`).
- If auto-detect fails, org and project are chosen explicitly with `sentry org list --json` and `sentry project list [org] --json`.
- Relevant environment variables are noted when they affect behavior: `SENTRY_ORG`, `SENTRY_PROJECT`, `SENTRY_DSN`, `SENTRY_URL`, `SENTRY_CLIENT_ID`.
- Time window, environment, release, and severity focus are stated.

## 2) Issue Discovery

- Pull unresolved or targeted issues with filters (`is:unresolved`, frequency, release, env, period).
- Record issue IDs and short IDs for all prioritized candidates.
- Capture count and user-impact indicators.
- Re-run with `--fresh` if results appear stale.

## 3) Evidence Collection

- Inspect target issue details with `sentry issue view`.
- Inspect at least one related event with `sentry event view`.
- Pull recent matching logs with `sentry log list` and inspect one entry via `sentry log view`.
- Inspect the relevant trace with `sentry trace view` or `sentry trace logs` when the problem spans services or performance is part of the failure.

## 4) Root Cause Analysis

- Run `sentry issue explain <issue-id>`.
- Validate Seer output against stack frames, events, logs, and traces.
- Record uncertainty if evidence is incomplete.

## 5) Remediation Plan

- Run `sentry issue plan <issue-id>` when explain is complete.
- Select explicit cause with `--cause` when multiple causes are returned.
- Convert the plan into implementation steps with owners and verification points.
- Use `sentry api` only when the CLI does not expose the needed read workflow directly.

## 6) Verification Strategy

- Define tests or checks that should fail before the fix and pass after.
- Define rollout monitoring (new issue count, error rate, user impact).
- Define rollback or mitigation if impact worsens.

## 7) Safety Checks

- Do not create projects, resolve issues, or mutate state without explicit user approval.
- Do not expose tokens, DSNs with secrets, or copied credentials in transcripts.

## 8) Reporting Standard

- Scope: org/project, issue IDs, filters, assumptions.
- Findings: key evidence lines from issue/event/log/trace analysis.
- Diagnosis: likely root cause and confidence.
- Actions: fix plan and validation commands.

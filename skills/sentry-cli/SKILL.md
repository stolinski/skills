---
name: sentry-cli
description: Use when working with the new Sentry CLI at cli.sentry.dev to authenticate, discover org and project context, inspect issues/events/logs/traces, run Seer explain/plan, or use sentry api.
---

# Sentry CLI

## Overview

This skill provides a repeatable workflow for debugging production errors with the official Sentry CLI (`sentry`) from `https://cli.sentry.dev`.

It keeps command usage aligned with the published CLI and the official packaged skill while adding agent-friendly defaults: prefer auto-detected scope first, collect evidence before diagnosis, use JSON output when results will be parsed, and keep state-changing commands explicit.

## When to Use

- You need to inspect Sentry issues, events, logs, or traces without opening the web UI first
- You need to authenticate the CLI or confirm which Sentry identity is active
- You need to discover organization, project, team, or repo context from a repo or monorepo
- You need Seer-generated root cause analysis and a fix plan
- You need JSON output for automation or agent pipelines
- You need direct REST API access through `sentry api`
- You need self-hosted Sentry CLI setup guidance

## When NOT to Use

- You are instrumenting SDK code in an app framework (use framework-specific docs)
- You need billing, quota, or account-admin workflows that are UI-only
- You need broad observability setup guidance beyond Sentry CLI operations

## Official Sources

- Docs: `https://cli.sentry.dev`
- Agent skill install: `npx skills add https://cli.sentry.dev`
- Repo: `https://github.com/getsentry/cli`

## Prerequisites

1. Install the CLI with any supported method:

```bash
curl https://cli.sentry.dev/install -fsS | bash
curl https://cli.sentry.dev/install -fsS | bash -s -- --version nightly
brew install getsentry/tools/sentry
npm install -g sentry
npx sentry@latest --help
```

2. Authenticate and verify identity:

```bash
sentry auth login
sentry auth status
sentry auth whoami --json
```

3. For Seer commands (`issue explain`, `issue plan`), ensure:
   - Seer is enabled for the organization
   - GitHub integration is connected
   - code mappings are configured

## Core Workflow

1. Confirm CLI readiness and auth

```bash
sentry auth status
sentry auth refresh
sentry org list --json
```

2. Resolve scope with auto-detection first
   - First try `sentry project view` or `sentry issue list` with no target so the CLI can use DSN and config discovery
   - If auto-detect fails, list orgs and projects explicitly and then choose a target
   - Respect `SENTRY_ORG`, `SENTRY_PROJECT`, and `SENTRY_DSN`; for self-hosted instances also use `SENTRY_URL` and `SENTRY_CLIENT_ID`

```bash
sentry project view --json
sentry issue list --json
sentry org list --json
sentry project list my-org --json
```

3. Collect evidence before diagnosing
   - Start with a filtered issue list and then inspect the target issue in detail
   - Pull at least one related event and log sample
   - Use traces when the failure is tied to latency, distributed systems, or cross-service flows

```bash
sentry issue list my-org/my-project --query "is:unresolved" --sort freq --period 14d --limit 20 --json
sentry issue view MYPROJ-ABC --json
sentry event view <event-id> --json
sentry log list my-org/my-project -q 'level:error' -n 100 --json
sentry trace view my-org/my-project <trace-id> --json
```

4. Run Seer only after you have basic evidence

```bash
sentry issue explain MYPROJ-ABC --json
sentry issue plan MYPROJ-ABC --cause 0 --json
```

5. Use the browser or direct API only when it helps
   - Use `-w` for a fast browser handoff on a specific resource
   - Use `sentry api` for unsupported endpoints, bulk reads, or advanced workflows
   - Keep mutating API calls opt-in and explicit

```bash
sentry issue view MYPROJ-ABC -w
sentry event view <event-id> -w
sentry api /projects/my-org/my-project/issues/ --paginate
```

6. Report findings in a fixed structure
   - Scope used
   - Evidence summary
   - Diagnosis and confidence
   - Action plan
   - Verification steps

## Agent-Oriented Principles

- Be explicit about scope early: org, project, issue ID, and time window
- Prefer project auto-detection first (`sentry project view` or `sentry issue list` with no target)
- Prefer `--json` for machine-readable output and deterministic parsing
- Prefer `--fresh` when cached results may be stale or a user asks for current state
- Prefer `-w` when a browser is the fastest way to inspect a single resource
- Collect evidence before diagnosis (issue view, event view, log sample, trace when relevant)
- Use Seer outputs as hypotheses; validate against stack frames, events, logs, and traces
- Run `sentry issue explain` before `sentry issue plan`
- Keep destructive or state-changing commands opt-in and explicit

## Common Patterns

### Fast unresolved sweep

```bash
sentry project view --json
sentry issue list my-org/my-project --query "is:unresolved" --sort freq --period 14d --limit 25 --json
sentry issue view MYPROJ-ABC --json
```

### Browser handoff for a specific resource

```bash
sentry issue view MYPROJ-ABC -w
sentry event view <event-id> -w
sentry log view my-org/my-project <log-id> -w
```

### Self-hosted auth

```bash
SENTRY_URL=https://sentry.example.com SENTRY_CLIENT_ID=your-client-id sentry auth login
SENTRY_URL=https://sentry.example.com sentry auth login --token YOUR_TOKEN
```

### Script-friendly extraction

```bash
sentry org list --json | jq '.[0]'
sentry issue list my-org/my-project --json | jq '.[0]'
sentry log list my-org/my-project --json | jq '.[] | select(.level == "error")'
```

## Anti-Patterns to Avoid

- Running `sentry issue plan` before `sentry issue explain`
- Using ambiguous project names in multi-org environments
- Assuming auto-detection failed permanently before checking env vars or using `--fresh`
- Omitting `--json` when output will be parsed by tools
- Treating Seer output as final truth without checking supporting evidence
- Skipping issue, event, or log evidence and jumping straight to Seer
- Exposing API tokens in logs, commits, or terminal transcripts
- Creating projects, resolving issues, or changing assignees without explicit user intent

## Output Contract

When using this skill, responses should include:

1. Scope used
   - org/project, issue IDs, filters, and assumptions
2. Evidence summary
   - key findings from issue, event, log, and/or trace commands
3. Diagnosis
   - likely root cause with confidence and affected surface area
4. Action plan
   - concrete fix steps and command-level follow-up
5. Verification
   - how to confirm the fix and monitor for regressions

## Prompt Templates

- "Investigate `MYPROJ-ABC` with Sentry CLI, summarize root cause, and propose a fix plan."
- "List unresolved high-frequency issues for `my-org/my-project` and rank by user impact."
- "Use `sentry api` to fetch all project issues, then summarize top recurring exceptions."
- "Explain this issue with Seer, then generate an implementation plan for cause 1."

## References

- [references/TRIAGE_CHECKLIST.md](references/TRIAGE_CHECKLIST.md)
- [references/COMMAND_PATTERNS.md](references/COMMAND_PATTERNS.md)

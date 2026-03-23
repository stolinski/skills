# Command Patterns

Common command patterns for Sentry CLI investigations.

## Setup

```bash
curl https://cli.sentry.dev/install -fsS | bash
curl https://cli.sentry.dev/install -fsS | bash -s -- --version nightly
brew install getsentry/tools/sentry
npm install -g sentry
npx sentry@latest --help
sentry auth login
sentry auth status
sentry auth whoami --json
sentry auth refresh
```

## Self-Hosted Setup

```bash
export SENTRY_URL=https://sentry.example.com
export SENTRY_CLIENT_ID=your-client-id

sentry auth login
SENTRY_URL=https://sentry.example.com sentry auth login --token YOUR_TOKEN
```

## Scope Discovery

```bash
sentry org list --json
sentry org view my-org --json
sentry project list --json
sentry project list my-org
sentry project view --json
sentry project view my-org/my-project
sentry team list my-org --json
```

## Issues

```bash
sentry issue list --json
sentry issue list my-org/my-project --limit 20 --json
sentry issue list my-org/my-project --query "is:unresolved" --sort freq --period 14d --limit 20 --json
sentry issue view MYPROJ-ABC --json
sentry issue view MYPROJ-ABC --spans all --json
sentry issue view MYPROJ-ABC -w
```

## Seer

```bash
sentry issue explain MYPROJ-ABC --json
sentry issue explain MYPROJ-ABC --force --json
sentry issue plan MYPROJ-ABC --json
sentry issue plan MYPROJ-ABC --cause 0 --json
```

## Events and Logs

```bash
sentry event view <event-id> --json
sentry event view <event-id> -w
sentry log list my-org/my-project -q 'level:error' -n 100 --json
sentry log list my-org/my-project -f -q 'database'
sentry log list my-org/my-project --trace <trace-id> --json
sentry log view my-org/my-project <log-id> --json
sentry log view my-org/my-project <log-id> -w
```

## Traces

```bash
sentry trace list my-org/my-project --sort duration --limit 20 --json
sentry trace view my-org/my-project <trace-id> --json
sentry trace view my-org/my-project <trace-id> -w
sentry trace logs my-org/my-project <trace-id> --period 24h --limit 100 --json
```

## API Escape Hatch

```bash
sentry api /organizations/
sentry api /projects/my-org/my-project/
sentry api /projects/my-org/my-project/issues/ --paginate
sentry api /issues/123456789/ --include

# Use only after explicit user confirmation
sentry api /issues/123456789/ --method PUT --field status=resolved
```

## JSON Pipelines

```bash
sentry org list --json | jq '.[].slug'
sentry issue list my-org/my-project --json | jq '.[0]'
sentry log list my-org/my-project --json | jq '.[] | select(.level == "error")'
sentry trace list my-org/my-project --json | jq '.[0]'
```

## Global Flags Worth Remembering

```bash
sentry issue list --verbose
sentry issue list --log-level debug
sentry issue list my-org/my-project --fresh --json
```

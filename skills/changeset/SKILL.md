---
name: changeset
description: Use when the user asks to create, generate, write, or apply a Changesets release note/version bump from current code changes (for example "make a changeset", "add a changeset", "apply changesets", or "version packages").
metadata:
  version: 1.0.0
---

# Changeset

Use this skill to create and optionally apply Changesets based on the code changes in the current session.

## What this skill does

- Detects what changed in the working tree.
- Maps changed files to affected packages in a monorepo or single-package repo.
- Proposes appropriate semver bump levels (`patch`, `minor`, `major`) with rationale.
- Creates a changeset markdown file in `.changeset/`.
- Applies pending changesets (`changeset version`) when requested.

## Preconditions

- Repository uses `@changesets/cli` (or has an equivalent `changeset` script).
- `.changeset/` exists or can be created.
- The user requested changeset creation and/or application.

## Safety rules

1. Do not apply changesets unless the user asked to apply them.
2. Do not invent package names; only use workspace package names found in `package.json` files.
3. Exclude private/internal packages unless the repo convention explicitly versions them.
4. If bump type is ambiguous, default to `patch` and explain why.
5. Never include unrelated files or speculative release notes.

## Workflow

1. Inspect repo state:
   - `git status --short`
   - `git diff --name-only` (and staged diff when relevant)
2. Detect package manager and scripts from root `package.json`.
3. Identify affected packages:
   - For monorepos, map changed files to nearest package root.
   - For single-package repos, use root package name.
4. Decide bump level per package:
   - `patch`: bug fixes, internal refactors, non-breaking behavior changes.
   - `minor`: new backwards-compatible functionality.
   - `major`: breaking changes.
5. Draft concise release note text focused on user impact.
6. Create changeset file in `.changeset/*.md` with frontmatter package bumps.
7. If user asked to apply it, run one of:
   - `pnpm changeset version`
   - `npm run changeset version` (if script exists)
   - `bunx changeset version`
8. Report exactly what was created/changed.

## Changeset template

```md
---
"@scope/package-a": minor
"@scope/package-b": patch
---

Short, concrete summary of what changed and why users should care.
```

## Command selection

Prefer project-native tooling in this order:

1. Existing script in `package.json` (for example `pnpm changeset`, `npm run changeset`).
2. Package-manager exec (`pnpm changeset`, `bunx changeset`, `npx changeset`).

## Output format

Return:

1. `Affected Packages`
2. `Bump Decisions`
3. `Changeset File`
4. `Applied` (yes/no, and command used)
5. `Notes` (ambiguities, assumptions, or follow-ups)

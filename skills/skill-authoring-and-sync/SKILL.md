---
name: skill-authoring-and-sync
description: Use when creating, editing, reviewing, or publishing agent skills, including quality standards, structure, examples, and syncing changes to the mini over SSH key auth.
---

# Skill Authoring and Sync

## Overview

This skill helps create high-quality skills that are clear, reusable, and maintainable.

It also includes the workflow for syncing OpenCode skill/config changes from MBP to mini using SSH key authentication.

## When to Use

- Creating a new skill from scratch
- Improving an existing skill's clarity and trigger quality
- Adding references, templates, and checklists to a skill
- Preparing a skill repo for publishing
- Syncing OpenCode settings/skills to the mini

## Skill Quality Standard

Every skill should include:

1. **Clear trigger scope**
   - Explicitly state when to use and when not to use
2. **Outcome focus**
   - Explain what successful output looks like
3. **Actionable workflow**
   - Step-by-step process, not generic advice
4. **Concrete examples**
   - Include ready-to-use snippets and patterns
5. **Safety and constraints**
   - Call out assumptions, risks, and anti-patterns

## Recommended Skill Structure

- Frontmatter: `name`, `description`
- Overview
- When to Use / When Not to Use
- Core principles and workflow
- Output contract (how responses should be structured)
- References (templates, checklists, examples)

Use template: [references/SKILL_SKELETON.md](references/SKILL_SKELETON.md)

## Authoring Workflow

1. Define a narrow job-to-be-done and explicit trigger phrases
2. Draft frontmatter and scope boundaries
3. Write operational steps (what to do first, then next)
4. Add references for repeated tasks (checklists/templates)
5. Validate readability and remove vague language
6. Test discoverability by listing installed/available skills
7. Sync changes to mini

Use checklist: [references/AUTHORING_CHECKLIST.md](references/AUTHORING_CHECKLIST.md)

## Sync Workflow (MBP -> Mini)

Prerequisite: SSH key auth works for `scotttolinski@mini`.

Push local OpenCode settings + skills:

```bash
~/.config/opencode/scripts/sync-opencode-settings.sh push scotttolinski@mini
```

Pull from mini back to MBP:

```bash
~/.config/opencode/scripts/sync-opencode-settings.sh pull scotttolinski@mini
```

Optional key override:

```bash
OPENCODE_SYNC_IDENTITY=~/.ssh/id_ed25519 ~/.config/opencode/scripts/sync-opencode-settings.sh push scotttolinski@mini
```

## Publishing Guidance

- Keep public READMEs client-neutral and machine-neutral
- Avoid personal infrastructure notes in public docs
- Put local environment specifics in private/internal docs or local automation scripts

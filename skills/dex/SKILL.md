---
name: dex
description: Use when tracking complex multi-step tasks, creating task hierarchies, maintaining persistent task state across sessions, building backlogs, or when the user explicitly asks to "use dex" for task management. Dex provides persistent memory for AI agents with GitHub/Shortcut sync capabilities.
---

# Dex - Persistent Task Tracking for AI Agents

Dex is a task tracking system that acts as a **master coordinator** for complex work—breaking down tasks, tracking progress, and preserving context across sessions. Unlike ephemeral session tracking, dex tasks are stored as files in the repository.

## Installation

```bash
npm install -g @zeeg/dex
# or
pnpm add -g @zeeg/dex
# or
bun add -g @zeeg/dex
```

## Core Concepts

- **Persistence**: Tasks survive beyond the current session as `.dex/` files in the repo
- **Context Preservation**: Decisions, progress, and rationale are captured
- **Hierarchy**: 3-level structure (Epic → Task → Subtask)
- **Dependencies**: Tasks can block other tasks
- **GitHub/Shortcut Sync**: Optional sync to Issues/Stories for permanent records

## Core Commands

### Dashboard & Listing

```bash
dex                    # Show dashboard (default command)
dex status             # Same as above - shows stats, ready tasks, blocked, completed
dex status --json      # Output as JSON

dex list               # List pending tasks in tree view
dex list --all         # Include completed tasks
dex list --ready       # Only unblocked tasks
dex list --blocked     # Only blocked tasks
dex list --in-progress # Only in-progress tasks
dex list --query "auth" # Search in name/description
dex list --flat        # Plain list instead of tree
```

### Creating Tasks

```bash
dex create "Task title"

dex create "Task title" --description "Full implementation details..."
dex create "Task title" --description "..." --priority 2
dex create "Task title" --blocked-by <other_task_id>
dex create "Subtask title" --parent <parent_task_id>
```

### Viewing & Editing

```bash
dex show <task_id>              # View full task details
dex show abc123 --expand        # Show ancestor descriptions
dex show abc123 --full          # No truncation
dex show abc123 --json          # JSON output

dex edit <task_id> --name "New name"
dex edit <task_id> --description "Updated context"
dex edit <task_id> --add-blocker <other_id>
dex edit <task_id> --remove-blocker <other_id>
```

### Status Management

```bash
dex start <task_id>             # Mark as in progress
dex start <task_id> --force     # Re-claim already in-progress task
dex complete <task_id> --result "What was accomplished"
dex complete <task_id> --result "..." --commit <sha>
dex complete <task_id> --result "..." --no-commit
dex delete <task_id>            # Delete task (and children)
dex archive <task_id>           # Archive completed task
```

### Planning

```bash
dex plan path/to/plan.md        # Create tasks from markdown plan
dex plan docs/SPEC.md --priority 2
dex plan feature.md --parent <id>
```

## GitHub Integration

```bash
dex sync                        # Sync all tasks to GitHub Issues
dex sync <task_id>              # Sync specific task
dex sync --dry-run              # Preview sync
dex sync --github               # GitHub only
dex sync --shortcut             # Shortcut only

dex import #42                  # Import GitHub issue
dex import --all                # Import all dex-labeled issues
dex import sc#123               # Import Shortcut story

dex export <task_id>            # Export to GitHub without sync
```

## When to Use Dex

**Use dex when:**
- Work spans multiple sessions or days
- Complex features need breakdown into steps
- Context and decisions must be preserved
- Work might be handed off or revisited
- Building a backlog before execution

**Skip dex when:**
- Quick, single-session task
- No context needs preserving
- Overhead isn't worth it

## Workflow for Agents

### Starting New Work

1. Create task with full context:
```bash
dex create "Add JWT authentication" --description "
## Goal
Implement JWT-based auth for the API.

## Requirements
- Token generation on login
- Verification middleware for protected routes
- Refresh token flow

## Files
- src/middleware/auth.ts
- src/routes/auth.ts

## Acceptance Criteria
- POST /api/auth/login returns JWT
- Protected routes return 401 without valid token
- All 24 tests passing
"
```

2. For complex work, break into subtasks:
```bash
dex create "Create auth middleware" --parent <parent_id>
dex create "Add login endpoint" --parent <parent_id>
dex create "Add token refresh flow" --parent <parent_id>
```

3. Mark as in progress:
```bash
dex start <task_id>
```

### During Implementation

```bash
dex list                        # Check current tasks
dex show <task_id>              # Review context
dex edit <task_id> --description "Updated: also need refresh tokens"
```

### Completing Tasks

```bash
dex complete <task_id> --result "
Added POST /api/auth/login endpoint.
Verifies credentials with bcrypt, returns JWT access token (15m)
and refresh token (7d). All 24 tests passing.
" --commit <sha>
```

## Writing Good Task Context

Include:
- **What** needs to be done (specific, not vague)
- **Why** it's needed (background, motivation)
- **How** to approach it (files to modify, patterns to follow)
- **Done when** (acceptance criteria)

**Good:**
> "Add rate limiting to /api/auth endpoints. Use express-rate-limit, 5 requests per minute per IP for /login, 20/min for /refresh. Return 429 with Retry-After header. Add to src/middleware/rate-limit.ts, apply in src/routes/auth.ts."

**Bad:**
> "Add rate limiting"

## Writing Good Results

Include:
- **What changed** (implementation summary)
- **Key decisions** (and why)
- **Verification** (tests passing, manual testing done)

**Good:**
> "Added rate limiting with express-rate-limit. Login: 5/min, refresh: 20/min. Returns 429 with Retry-After header. Added 6 tests for rate limit scenarios. All 30 tests passing."

**Bad:**
> "Added rate limiting"

## Task Hierarchy

- **Epic**: Large initiatives (5+ tasks). Example: "Add user authentication system"
- **Task**: Significant work items. Example: "Implement JWT middleware"
- **Subtask**: Atomic steps. Example: "Add token verification function"

Maximum depth is 3 levels.

## Behavior Notes

- Subtasks must complete before parent can be completed
- Blocked tasks CAN be completed (warning shown) for flexibility
- Deleting a parent deletes all children
- Use GitHub issue numbers (permanent) in commits, not task IDs (ephemeral)

## Configuration

```bash
dex init                        # Interactive setup
dex init -y                     # Accept defaults
dex dir                         # Print task storage path
dex dir --global                # Print global config path
dex config --list               # Show all config values
dex config sync.github.enabled=true  # Enable GitHub sync
dex doctor                      # Check for issues
dex doctor --fix                # Check and fix issues
```

## Session Handoff

When ending a session with incomplete work:

```bash
dex edit <task_id> --description "
## Progress
- Completed: JWT middleware added
- In Progress: Login endpoint (70% done)
- Blocked: Need refresh token strategy

## Next Steps
- Finish login endpoint validation
- Add refresh token rotation
"
```

Keep task in_progress so next session knows to continue.

# Code Quality Audit Tracking

## Session summary

- Started: YYYY-MM-DD
- Last updated: YYYY-MM-DD
- Repository scope: <paths or package names>
- Current phase: <audit-only|safe-remediation>

## Rating legend

| Total % | Rating | Priority |
|---|---|---|
| 87-100% | Excellent | No action needed |
| 70-86% | Good | Low priority improvements |
| 50-69% | Adequate | Medium priority |
| 30-49% | Weak | High priority |
| < 30% | Insufficient | Critical |

## File entries

### `<path/to/file.ts>`

- status: `analyzed|updated|blocked`
- score: `<NN%>` (`Excellent|Good|Adequate|Weak|Insufficient`)
- placement_status: `correct|moved|deferred`
- import_impact: `low|medium|high`
- move_reason: `<contract violation or N/A>`
- moved_from: `<old path or N/A>`
- moved_to: `<new path or N/A>`

#### Findings

- `[severity: high|medium|low] [confidence: high|medium|low] <path:line> - <finding>`

#### Improvements applied

- `<concrete change and why>`

#### Open questions

- `<ambiguity or design decision needed>`

#### Verification

- `lint:fix`: `pass|fail|not-run`
- `typecheck`: `pass|fail|not-run`
- `build`: `pass|fail|not-run`
- `test`: `pass|fail|not-run`
- `test:e2e`: `pass|fail|not-run`

#### Final disposition

- `<kept|reverted|deferred>`

---

### `<path/to/next-file.ts>`

- Repeat the same structure.

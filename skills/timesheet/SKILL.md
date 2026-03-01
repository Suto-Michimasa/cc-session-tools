---
name: timesheet
description: Estimate per-project work time from Claude Code session timestamps
user_invocable: true
arguments: "YYYY-MM-DD..YYYY-MM-DD or YYYY-Www or YYYY-MM-DD (optional, default: current week)"
---

Estimate work time per project based on Claude Code session timestamps in `~/.claude/projects/`.

## Procedure

### 1. Determine target period

Parse `$ARGUMENTS`:
- If empty: current ISO week (Monday through today)
- If `YYYY-Www` (e.g. `2026-W09`): that ISO week (Monday to Sunday)
- If `YYYY-MM-DD..YYYY-MM-DD`: that date range
- If `YYYY-MM-DD`: that single day
- If `YYYY-MM`: that entire month

Calculate `period_start` and `period_end` as YYYY-MM-DD strings.

### 2. Load template

1. `templates/ja.md` in this skill's directory (for Japanese environment)
2. Fallback: `templates/default.md`

Replace `{{period_label}}`, `{{period_start}}`, `{{period_end}}` with computed values.

### 3. Calculate work time

For each `.jsonl` file in `~/.claude/projects/*/`:

1. Read the first line — skip if `type` is `"queue-operation"` (subagent session).
2. Collect all timestamps from lines whose date falls within the target period.
3. Sort timestamps chronologically.
4. Split into work blocks: if the gap between consecutive timestamps exceeds **30 minutes** (idle threshold), start a new block.
5. Sum block durations. Round to the nearest **15 minutes** (minimum 15 minutes if any user messages exist).
6. Count user messages and active dates.

Group results by project directory.

### 4. Generate report

Follow the template's HTML comments as instructions. Output directly to the conversation. Do NOT write to a file.

Populate:
- `{{period_label}}`, `{{period_start}}`, `{{period_end}}` — period identifiers
- `{{total_hours}}`, `{{total_sessions}}`, `{{project_count}}` — totals
- Per-project rows: `{project_name}`, `{hours}`, `{percentage}`, `{session_count}`, `{prompt_count}`, `{active_days}`

Sort projects by hours descending.

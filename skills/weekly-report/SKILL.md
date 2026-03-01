---
name: weekly-report
description: Generate a weekly summary report from daily reports or session history
user_invocable: true
arguments: "YYYY-Www or YYYY-MM-DD..YYYY-MM-DD (optional, default: current week)"
---

Generate a Markdown weekly report by aggregating daily reports or session data from `~/.claude/projects/`.

## Procedure

### 1. Load config

Read `config.json` bundled with this skill. Defaults: `dailyReportDir: "~/daily-reports"`, `outputDir: "~/weekly-reports"`, `templatePath: ""`, `language: "en"`.

### 2. Determine target week

Parse `$ARGUMENTS`:
- If empty: current ISO week (Monday through Sunday containing today)
- If `YYYY-Www` (e.g. `2026-W09`): that ISO week
- If `YYYY-MM-DD..YYYY-MM-DD`: that date range
- If single `YYYY-MM-DD`: the ISO week containing that date

Calculate `week_start` (Monday) and `week_end` (Sunday). Derive `week_label` as ISO week identifier (e.g. "2026-W09").

### 3. Load template

1. `config.templatePath` if set and exists
2. Otherwise `templates/{config.language}.md` in this skill's directory
3. Fallback: `templates/default.md`

Replace `{{week_label}}`, `{{week_start}}`, `{{week_end}}` with computed values.

### 4. Gather daily data

For each date in the range, check if `config.dailyReportDir/YYYY-MM-DD.md` exists.

- **If daily reports exist**: Read and parse them. Extract highlights, tasks by project, learnings, and next actions from each day's report.
- **If daily reports are missing**: Fall back to direct session data extraction using the same approach as the daily-report skill — grep for the date in `~/.claude/projects/*/*.jsonl`, skip subagent sessions (first line `type:"queue-operation"`), collect user prompts and tool calls.

### 5. Collect session statistics

For each date in the range, scan `~/.claude/projects/*/*.jsonl` for sessions active on that date. Collect per-project metrics: session count, prompt count, active days.

### 6. Generate report

Follow the template's HTML comments as instructions. Populate:
- `{{week_label}}`, `{{week_start}}`, `{{week_end}}` — week identifiers
- Per-project blocks: `{project_name}`, `{session_count}`, `{prompt_count}`, `{active_days}`

Key synthesis rules:
- Consolidate daily highlights into weekly achievements (group by theme, not by day)
- Merge daily tasks into week-level progress (deduplicate across days)
- Aggregate learnings, remove duplicates
- Derive next-week priorities from in-progress tasks and unstarted items

### 7. Write output

Write to `config.outputDir/YYYY-Www.md` (e.g. `~/weekly-reports/2026-W09.md`). Create the directory with `mkdir -p` if needed.

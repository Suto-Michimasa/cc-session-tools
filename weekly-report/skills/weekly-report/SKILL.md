---
name: weekly-report
description: Generate a weekly summary report from daily reports or session history
user_invocable: true
argument-hint: [week-or-range]
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

### 4. Gather daily reports

For each date in the target week, check if `config.dailyReportDir/YYYY-MM-DD.md` exists. Read each existing file in its entirety. The content will be used as-is for synthesis in Step 6 — do not attempt to parse specific sections or headings from the daily reports, as the format may vary.

### 5. Collect session statistics

Gather per-project metrics for the entire target week from session data:

```bash
grep -rl 'YYYY-MM-DD' ~/.claude/projects/*/*.jsonl
```

Run for each date in the target week. Group results by project directory. For each project, count sessions (files) and user messages (`"type":"user"`). Skip subagent sessions (first line `type:"queue-operation"`). Count the number of distinct dates each project appears on as `{active_days}`.

This provides `{project_name}`, `{session_count}`, `{prompt_count}`, `{active_days}` for each project regardless of whether daily reports exist.

### 6. Generate report

Follow the template's HTML comments as instructions.

**Input**: The collected daily report contents (from Step 4) and session statistics (from Step 5).

**Populate placeholders**:
- `{{week_label}}`, `{{week_start}}`, `{{week_end}}` — week identifiers
- Per-project blocks: `{project_name}`, `{session_count}`, `{prompt_count}`, `{active_days}`

**Synthesis rules**:
- Read all daily reports holistically, then synthesize a week-level view.
- For dates without daily reports, use the session statistics to note which projects were active (project names and prompt counts give a rough sense of effort).
- Achievements: consolidate by theme/project, not chronologically.
- Project activity: merge and deduplicate across days.
- Learnings: deduplicate, keep only the most valuable.
- Next week priorities: derive from the most recent daily report's forward-looking items plus any unfinished work.

### 7. Write output

Write to `config.outputDir/YYYY-Www.md` (e.g. `~/weekly-reports/2026-W09.md`). Create the directory with `mkdir -p` if needed.

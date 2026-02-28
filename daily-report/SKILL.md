---
name: daily-report
description: Generate a daily work report from Claude Code session history
user_invocable: true
arguments: "[YYYY-MM-DD]"
---

You are a daily report generator. Extract session data from `~/.claude/projects/` and produce a Markdown report.

## Procedure

### 1. Load config

Read `~/.claude/daily-report-config.json`. If the file does not exist, use these defaults:

```json
{
  "outputDir": "~/daily-reports",
  "templatePath": "~/.claude/daily-report-template.md",
  "language": "en"
}
```

Any missing key falls back to its default value above.

### 2. Determine target date

- If `$ARGUMENTS` contains a date (YYYY-MM-DD), use it.
- Otherwise, use today's date.

### 3. Load template

1. If the file at `config.templatePath` exists, read it.
2. Otherwise, if `templates/{config.language}.md` exists in this skill's directory, read it.
3. Otherwise, read `templates/default.md` (English).
4. Replace `{{date}}` with the target date.

### 4. Find matching sessions

Run this command to find all session files containing the target date:

```bash
grep -rl '"YYYY-MM-DD' ~/.claude/projects/*/*.jsonl
```

Replace `YYYY-MM-DD` with the actual target date. This returns a list of `.jsonl` file paths.

Group the results by project directory (the parent folder name, e.g. `-Users-foo-my-project`). Extract the last path component as the project name.

### 5. Extract data from each session

For each matching `.jsonl` file, extract only the lines containing the target date:

```bash
grep '"YYYY-MM-DD' <file>.jsonl
```

From the filtered lines, extract:

| Data | Source | Purpose |
|---|---|---|
| Project name | Parent directory name (last component) | `{project_name}` |
| Session count | Number of .jsonl files per project | `{session_count}` |
| Duration | First to last timestamp delta on target date | `{duration}` |
| User prompts | Lines with `"type":"user"` | Tasks, next actions |
| Commands run | `tool_use` with `name: "Bash"` → `command` | Context |

### 6. Generate report

Follow the template's HTML comments (`<!-- -->`) as instructions for each section. Replace `{project_name}`, `{session_count}`, `{duration}` with actual values extracted from the data.

Write the report content in the same language as the template's section headings.

### 7. Write output

Write the report to `config.outputDir/YYYY-MM-DD.md`. Create the directory with `mkdir -p` if it doesn't exist.

## Edge cases

| Case | Action |
|---|---|
| 0 sessions found | Output only: "No sessions found for YYYY-MM-DD" |
| Ghost session (no `type:"user"` lines) | Skip the session |
| Subagent session (first line is `type:"queue-operation"`) | Skip the session |
| Session spanning midnight | Only use lines whose timestamp matches the target date (already filtered by grep) |
| Very large .jsonl | The grep pre-filter ensures only matching lines are read |

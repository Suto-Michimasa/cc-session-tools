---
name: daily-report
description: Generate a daily work report from Claude Code session history
user_invocable: true
arguments: "YYYY-MM-DD (optional)"
---

Generate a Markdown daily report from `~/.claude/projects/` session history.

## Procedure

### 1. Load config

Read `config.json` bundled with this skill. Defaults: `outputDir: "~/daily-reports"`, `templatePath: ""`, `language: "en"`.

### 2. Determine target date

Use `$ARGUMENTS` as YYYY-MM-DD if provided, otherwise today.

### 3. Load template

1. `config.templatePath` if set and exists
2. Otherwise `templates/{config.language}.md` in this skill's directory
3. Fallback: `templates/default.md`

Replace `{{date}}` with the target date.

### 4. Extract session data

```bash
grep -rl '"YYYY-MM-DD' ~/.claude/projects/*/*.jsonl
```

Group results by project directory. For each matching `.jsonl`, extract lines containing the target date via `grep`. From those lines, collect user prompts (`"type":"user"`) and tool calls. Skip sessions with no user messages or whose first line is `type:"queue-operation"` (subagents). Count the number of user messages per session as `{prompt_count}`.

### 5. Generate report

Follow the template's HTML comments (`<!-- -->`) as instructions for each section. Replace `{project_name}`, `{session_count}`, `{prompt_count}` with values from the data.

### 6. Write output

Write to `config.outputDir/YYYY-MM-DD.md`. Create the directory with `mkdir -p` if needed.

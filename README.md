# cc-session-tools

A Claude Code plugin marketplace for productivity tracking using session history.

It scans `~/.claude/projects/` for session data and provides daily/weekly reports, session search, and timesheet estimation. Install all plugins at once or pick only what you need.

## Install

```bash
# Add the marketplace
/plugin marketplace add Suto-Michimasa/cc-session-tools

# Install all plugins
/plugin install daily-report@session-tools
/plugin install weekly-report@session-tools
/plugin install session-search@session-tools
/plugin install timesheet@session-tools

# Or install only what you need
/plugin install daily-report@session-tools
```

<details>
<summary>Manual install (without plugin system)</summary>

```bash
git clone https://github.com/Suto-Michimasa/cc-session-tools.git
# Install all
cp -r cc-session-tools/*/skills/* ~/.claude/skills/

# Or install individually
cp -r cc-session-tools/daily-report/skills/daily-report ~/.claude/skills/
```

</details>

## Skills

### `/daily-report` — Daily work report

Generate a structured daily report from session history.

```bash
> /daily-report
> /daily-report 2025-06-15
```

Reports are saved to `~/daily-reports/YYYY-MM-DD.md` by default. Edit `config.json` to customize output directory, language, and template.

<details>
<summary>Example output</summary>

```markdown
# Daily Report — 2025-06-15

## Highlights
- Dashboard loading time reduced from 3.2s to 0.8s by eliminating N+1 queries

## Tasks by Project

### team-dashboard — 3 sessions, 24 prompts
- [x] Investigate and fix slow dashboard loading
- [-] Add real-time updates via WebSocket (remaining: reconnection logic)

## Learnings
- PostgreSQL's `EXISTS` subquery outperforms `JOIN + DISTINCT` for "has any" checks

## Next Actions
- Implement WebSocket reconnection with exponential backoff
```

</details>

### `/weekly-report` — Weekly summary

Aggregate daily reports into a weekly summary for 1on1 or team meetings.

```bash
> /weekly-report
> /weekly-report 2026-W09
> /weekly-report 2026-02-24..2026-02-28
```

Reads existing daily reports if available, otherwise falls back to raw session data. Reports are saved to `~/weekly-reports/YYYY-Www.md`.

### `/session-search` — Search session history

Search across all sessions by keyword.

```bash
> /session-search "N+1 query fix"
> /session-search migration
```

Results are displayed in the conversation, grouped by project with surrounding context.

### `/timesheet` — Work time estimation

Estimate per-project work time from session timestamps.

```bash
> /timesheet
> /timesheet 2026-02-14
> /timesheet 2026-W09
> /timesheet 2026-02-01..2026-02-28
```

Calculates time based on message timestamps, splitting work blocks at 30-minute idle gaps and rounding to 15-minute increments.

## Configuration

`daily-report` and `weekly-report` have `config.json` files in their skill directories. `session-search` and `timesheet` have no configuration (sensible defaults are built-in).

### daily-report config

| Key | Default | Description |
|---|---|---|
| `outputDir` | `~/daily-reports` | Directory to save generated reports |
| `templatePath` | `""` | Path to custom template (empty = use bundled template) |
| `language` | `en` | Template language — selects `templates/{language}.md` |

### weekly-report config

| Key | Default | Description |
|---|---|---|
| `dailyReportDir` | `~/daily-reports` | Where to look for existing daily reports |
| `outputDir` | `~/weekly-reports` | Directory to save weekly reports |
| `templatePath` | `""` | Path to custom template |
| `language` | `en` | Template language |

## Custom templates

Templates use YAML frontmatter + Markdown with `<!-- -->` instruction comments. To customize:

```bash
cp cc-session-tools/daily-report/skills/daily-report/templates/default.md ~/.claude/daily-report-template.md
```

Then set `templatePath` in the corresponding `config.json`.

## License

MIT

# cc-daily-report

A Claude Code skill that generates daily work reports from your session history.

It scans `~/.claude/projects/` for sessions on the target date, extracts what you worked on, and produces a structured Markdown report.

## Install

```bash
git clone https://github.com/Suto-michimasa/cc-daily-report.git
cp -r cc-daily-report/daily-report ~/.claude/skills/daily-report
```

This copies the skill, config, and all templates at once. Edit `~/.claude/skills/daily-report/config.json` to customize output directory, language, etc.

## Usage

```bash
# In Claude Code — generate today's report
> /daily-report

# Specify a date
> /daily-report 2025-06-15

# Headless
claude -p "/daily-report" --output-format text > ~/daily-reports/$(date +%Y-%m-%d).md
```

Reports are saved to `~/daily-reports/YYYY-MM-DD.md` by default.

## Example output

```markdown
# Daily Report — 2025-06-15

## 🏆 Today's Highlights
- Dashboard loading time reduced from 3.2s to 0.8s by eliminating N+1 queries
- Designed and got team approval on the notification system API spec

## 📋 Tasks by Project

### team-dashboard — 3 sessions, 24 prompts

- [x] Investigate and fix slow dashboard loading (N+1 query in user list endpoint)
- [x] Review PR #248 — notification preference migration
- [-] Add real-time updates via WebSocket (remaining: reconnection logic)

### personal-blog — 1 session, 8 prompts

- [x] RSS feed generation with full-text content
- [-] Dark mode toggle (remaining: system preference detection)
- [ ] OGP image auto-generation (blocked: need to choose a library)

## 💡 Learnings
- PostgreSQL's `EXISTS` subquery outperforms `JOIN + DISTINCT` for "has any" checks — reduced the query from 1.2s to 40ms
- When designing notification APIs, separating delivery channel (email/push/in-app) from notification type keeps the schema flexible

## ⏭️ Next Actions
- Implement WebSocket reconnection with exponential backoff (team-dashboard)
- Choose between @vercel/og and satori for OGP image generation (personal-blog)
- Prepare demo for Wednesday's team sync
```

## Configuration

Edit `~/.claude/skills/daily-report/config.json`:

```json
{
  "outputDir": "~/daily-reports",
  "templatePath": "",
  "language": "en"
}
```

| Key | Default | Description |
|---|---|---|
| `outputDir` | `~/daily-reports` | Directory to save generated reports |
| `templatePath` | `""` | Path to custom template (empty = use bundled template) |
| `language` | `en` | Template language — selects `templates/{language}.md` when templatePath is empty |

All keys are optional. Missing keys fall back to defaults.

## Custom templates

The default template outputs:

- **Highlights** — Top achievements of the day
- **Tasks by Project** — TODO-style progress per project (`[x]` done, `[-]` in progress, `[ ]` blocked)
- **Learnings** — Technical insights and discoveries
- **Next Actions** — What to do tomorrow

To use a custom template:

```bash
cp cc-daily-report/daily-report/templates/default.md ~/.claude/daily-report-template.md
```

Create your own template following the same format (YAML frontmatter + Markdown with `<!-- -->` instructions for Claude).

## License

MIT

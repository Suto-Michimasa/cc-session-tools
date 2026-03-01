---
name: default
description: Timesheet - work time estimate
---

# Timesheet — {{period_label}}

> Estimated from Claude Code session timestamps ({{period_start}} ~ {{period_end}})
> Idle threshold: gaps > 30min split into separate work blocks. Rounded to 15min.

## Summary

- **Total estimated time**: {{total_hours}} hours
- **Total sessions**: {{total_sessions}}
- **Projects**: {{project_count}}

## Time by Project

<!--
Render as a Markdown table. Sort by hours descending.
The percentage column shows each project's share of total time.
-->

| Project | Hours | % | Sessions | Prompts | Active Days |
|---------|------:|--:|---------:|--------:|------------:|
| {project_name} | {hours} | {percentage}% | {session_count} | {prompt_count} | {active_days} |

## Daily Breakdown

<!--
If the period spans multiple days, show a per-day table:
| Date | Project | Hours | Sessions |
Only include this section for multi-day periods. Omit for single-day timesheets.
-->

---

*Time estimates are based on message timestamps with idle gaps (>30min) excluded. Actual work time may differ.*

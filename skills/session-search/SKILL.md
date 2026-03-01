---
name: session-search
description: Search across all Claude Code sessions by keyword
user_invocable: true
arguments: "search query (required)"
---

Search across all Claude Code session history in `~/.claude/projects/` and display matching results grouped by project.

## Procedure

### 1. Validate arguments

`$ARGUMENTS` is required. If empty, print usage and stop:

```
Usage: /session-search <query>
Example: /session-search "N+1 クエリ"
```

Extract the search query. Remove surrounding quotes if present.

### 2. Find matching session files

```bash
grep -Frl '$QUERY' ~/.claude/projects/*/*.jsonl 2>/dev/null
```

Use `-F` for fixed-string matching (no regex). If no matches, report "No results found for: $QUERY" and stop.

### 3. Extract matching messages

For each matching file:

1. Read the first line — skip if `type` is `"queue-operation"` (subagent session).
2. Parse all lines as JSON. Collect messages with their role, content, and timestamp.
3. Find user messages (`"type":"human"` or `"role":"user"`) whose content contains the query (case-insensitive).
4. For each match, capture 1 surrounding message before and after for context.
5. Stop after 20 total matches across all files.

Content may be a string or an array of `{"type":"text","text":"..."}` objects — handle both.

### 4. Format and display results

Output results as Markdown directly to the conversation. Do NOT write to a file.

Group results by project. For each match:

```markdown
## Search Results for: "$QUERY"

Found N matches across M projects.

### project-name

**YYYY-MM-DD HH:MM** (session abcd1234)
> matched message content (truncated to 300 chars)...
>
> _context: surrounding message excerpt..._
```

Bold the matching query text within the displayed content. Truncate long messages to 300 characters.

### 5. Print summary

```
---
Searched N sessions across M projects. Showing top K results.
```

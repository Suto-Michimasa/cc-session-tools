---
name: session-search
description: Search across all Claude Code sessions by keyword
user_invocable: true
argument-hint: [query]
---

Search across all Claude Code session history in `~/.claude/projects/` using grep-based agentic search.

**Prerequisite**: Python 3 (pre-installed on macOS and major Linux distros).

## JSONL structure

Each session file is a JSONL where each line is a JSON object. User messages look like:

```jsonl
{"type":"user","message":{"role":"user","content":"search query here"},"uuid":"...","timestamp":"2025-02-14T10:30:00.000Z",...}
```

Key fields:
- `type`: `"user"` for user messages, `"assistant"` for responses
- `message.content`: string, or array of `{"type":"text","text":"..."}` objects
- `timestamp`: ISO 8601 format
- First line `type: "queue-operation"` indicates a subagent session (skip these)

## Procedure

### 1. Validate arguments

`$ARGUMENTS` is required. If empty, print usage and stop:

```
Usage: /session-search <query>
Example: /session-search "Slack API integration"
Example: /session-search "N+1 query optimization"
```

Extract the search query. Remove surrounding quotes if present.

### 2. Generate keywords

Interpret the user's query and generate **3-5 search keywords**.

- Include the original terms if concrete enough.
- Add synonyms and related technical terms.
- If the query is in a non-English language, include both the original language and English terms.
- Do NOT overthink — start simple.

Print the keywords:
```
Searching with: [keyword1, keyword2, keyword3, ...]
```

### 3. Search, score, and extract

Use a **single** `grep -Erl` with regex OR to find all candidate files in one scan:

```bash
grep -Erl 'keyword1|keyword2|keyword3' ~/.claude/projects/*/*.jsonl 2>/dev/null
```

This is 2-7x faster than running separate `grep -Frl` per keyword.

Then, use a **Python script** to score, extract, and format results in one pass. The script should:

1. **Score** each candidate file by how many distinct keywords it contains.
2. **Skip** subagent sessions (first line has `"type":"queue-operation"`).
3. **Parse** each line as JSON. Find user messages where `type` is `"user"`.
4. **Filter noise** — skip messages where `message.content` is longer than 2000 characters (skill/system prompt expansions).
5. **Extract** the first matching user message per session, along with its `timestamp`.
6. **Capture context** — also extract the next `"assistant"` message content (truncated to 200 chars) as context.

Process files in score order (highest first). Stop after **5 results**.

If zero results, try **one more search** with alternative/broader keywords.

### 4. Display results

Output as Markdown directly to the conversation:

```markdown
## Search Results for: "$ORIGINAL_QUERY"

Keywords: [keyword1, keyword2, ...]

### project-name

**YYYY-MM-DD HH:MM** `session: full-uuid-here`
> matched message content (truncated to 300 chars)...
>
> _context: assistant response excerpt (truncated to 200 chars)..._

### another-project

**YYYY-MM-DD HH:MM** `session: full-uuid-here`
> matched message content...
>
> _context: assistant response excerpt..._

---
Found N results across M projects.
```

- Derive **project name** from the directory name (strip the encoded home path prefix).
- Use the **full UUID** from the filename (without `.jsonl`) as session ID.
- Format timestamp as `YYYY-MM-DD HH:MM` in local timezone.
- Bold the matching keywords in displayed content.
- Truncate user messages to 300 characters, assistant context to 200 characters.

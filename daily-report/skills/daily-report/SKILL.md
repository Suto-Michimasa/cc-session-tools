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

Two-pass approach: first identify which projects had activity, then extract details only for active projects.

**Pass 1 — Discover active projects (single command):**

```bash
grep -rl 'YYYY-MM-DD' ~/.claude/projects/*/*.jsonl | python3 -c "
import sys, os, json
files = [l.strip() for l in sys.stdin if l.strip()]
projects = {}
for f in files:
    first = json.loads(open(f).readline())
    if first.get('type') == 'queue-operation': continue
    proj = os.path.basename(os.path.dirname(f))
    projects.setdefault(proj, []).append(f)
for proj, fs in sorted(projects.items()):
    print(f'{proj} ({len(fs)} files): {\" \".join(fs)}')
"
```

Review the output. Decide which projects are relevant (skip projects with only non-date-related matches if obvious). Then proceed to Pass 2 for each project.

**Pass 2 — Extract user messages per project (one command per project):**

Run for each project's files together:

```bash
cat <file1> <file2> ... | grep 'YYYY-MM-DD' | python3 -c "
import sys, json
msgs = []
for l in sys.stdin:
    try:
        d = json.loads(l)
    except: continue
    if d.get('type') not in ('human','user'): continue
    c = d.get('message',{}).get('content','')
    if isinstance(c, list):
        c = ' '.join(x.get('text','') for x in c if isinstance(x,dict) and x.get('type')=='text')
    if c and not c.startswith('(local command'):
        msgs.append(c[:150])
print(f'PROMPT_COUNT: {len(msgs)}')
for i,m in enumerate(msgs): print(f'[{i+1}] {m}')
"
```

Derive `{project_name}` from directory name. `{session_count}` = number of files per project. Skip projects with 0 user messages.

### 5. Generate report

Follow the template's HTML comments (`<!-- -->`) as instructions for each section. Replace `{project_name}`, `{session_count}`, `{prompt_count}` with values from the data.

### 6. Write output

Write to `config.outputDir/YYYY-MM-DD.md`. Create the directory with `mkdir -p` if needed.

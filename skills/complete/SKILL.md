---
name: complete
description: Complete a task and generate a final usage report.
---

# usage:complete

Complete a tracked task and generate a final usage report saved to local storage.

## Trigger

- "usage complete"
- "usage done"
- "작업 추적 종료"

## Parameters

- **taskId** (optional): Task to complete. If omitted and only one active task exists, uses that one. If multiple, ask user.
- **summary** (optional): Brief description of what was accomplished.

## Procedure

### 1. Identify task

```bash
cat "$HOME/.claude/usage-tracker/tasks.json"
```

Find the target task. If ambiguous, list active tasks and ask.

### 2. Get final usage data

Same as snap — query ccusage and aggregate.

```bash
ccusage session --since {task.startedAt as YYYYMMDD} --json --breakdown
```

### 3. Generate report

Create markdown report at `~/.claude/usage-tracker/reports/{taskId}-{date}.md`:

```markdown
# Usage Report: {label}

## Summary

- **Task**: {label}
- **Approach**: {approach}
- **Period**: {startedAt} ~ {completedAt}
- **Duration**: {days}d
- **Tags**: {tags}

## Token Usage

| Metric | Value |
|--------|-------|
| Total Cost | ${cost} |
| Total Tokens | {tokens} |
| Input Tokens | {input} |
| Output Tokens | {output} |
| Cache Read | {cacheRead} |
| Cache Create | {cacheCreate} |

## Model Breakdown

| Model | Cost | Tokens |
|-------|------|--------|
| {model} | ${cost} | {tokens} |

## Sessions

| # | Session ID | Added | Auto |
|---|-----------|-------|------|
| 1 | {id} | {date} | {yes/no} |

## Efficiency Metrics

- **Cost per day**: ${cost/days}
- **Output tokens per session**: {output/sessions}
- **Cache hit ratio**: {cacheRead / (cacheRead + cacheCreate + input)}%

## Notes

{user summary if provided}
```

### 4. Update tasks.json

Set task status to "completed", add completedAt timestamp.

### 5. Output

```
[Usage Tracker] Task completed
- Report saved: ~/.claude/usage-tracker/reports/{taskId}-{date}.md
- Total cost: ${cost}
- Duration: {days}d
- Sessions: {count}
```

Print the full report content to the user as well.

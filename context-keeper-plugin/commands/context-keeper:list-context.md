---
name: context-keeper:list-context
description: List all saved contexts across all sessions, or filter by session
argument-hint: "[session-id]"
---

# List Context Command

Display saved context summaries from the context-keeper plugin.

## Arguments

- `$ARGUMENTS` - Optional session ID to filter contexts. If omitted, lists ALL individual contexts across ALL sessions.

## Instructions

When this command is invoked:

1. **Read the summaries index** at `{project}/.claude/summaries/index.json`
   - If it doesn't exist, inform the user no summaries are available

2. **If no session ID provided** (list all contexts):
   - Show ALL individual context summaries across ALL sessions
   - Display each context as a separate row (not grouped)
   - Sort by most recent first
   - Include session ID, timestamp, trigger, files modified, message count

3. **If session ID provided** (`$ARGUMENTS` is not empty):
   - Filter summaries to show only entries matching that session ID
   - Show all compaction timestamps for that session
   - Display detailed information for each compaction

## Output Format

### When listing all contexts (no argument):

```
## All Saved Contexts

| # | Session ID | Timestamp | Trigger | Files | Messages | Topics |
|---|------------|-----------|---------|-------|----------|--------|
| 1 | abc123...  | 2025-11-24 19:04 | auto | 15 | 150 | #api, #auth |
| 2 | abc123...  | 2025-11-24 15:30 | manual | 8 | 95 | #refactor |
| 3 | def456...  | 2025-11-23 14:30 | auto | 3 | 45 | #bugfix |
| 4 | ghi789...  | 2025-11-22 10:15 | manual | 12 | 200 | #feature |

Total: 4 context summaries across 3 sessions

Use `/context-keeper:load-context <session-id>` to load a specific context.
Use `/context-keeper:list-context <session-id>` to see details for one session.
```

### When listing specific session:

```
## Context History for Session abc123...

### Compaction 1: 2025-11-24 19:04:48
- **Trigger:** manual
- **Files Modified:** 15
- **Messages:** 150
- **Topics:** #authentication, #api, #bugfix
- **Summary Path:** abc123.../20251124_190448/summary.md

### Compaction 2: 2025-11-24 15:30:22
- **Trigger:** auto
- **Files Modified:** 8
- **Messages:** 95
- **Topics:** #refactoring, #tests
- **Summary Path:** abc123.../20251124_153022/summary.md

Would you like me to load one of these contexts?
```

## Implementation

Use the Read tool to:
1. Read `.claude/summaries/index.json` for the summary index
2. Parse and format the results

### Index Structure

```json
{
  "summaries": [
    {
      "session_id": "abc123...",
      "timestamp": "20251124_190448",
      "created_at": "2025-11-24T19:04:48Z",
      "trigger": "manual",
      "project": "/path/to/project",
      "files_modified": ["file1.py", "file2.ts"],
      "message_count": 150,
      "summary_path": "abc123.../20251124_190448/summary.md"
    }
  ],
  "last_session": "abc123..."
}
```

## Error Handling

- **No summaries directory**: "No context summaries found. Run `/compact` to create your first summary."
- **No index.json**: "Summary index not found. Context summaries will be created automatically during compaction."
- **Session not found**: "No contexts found for session '{id}'. Available sessions: [list first 5]"

## Related Commands

- `/context-keeper:list-sessions` - List all stored sessions
- `/context-keeper:load-context` - Load a specific context summary

---
name: recall
description: "Three-layer memory with auto-fix: Core (essential), Working (auto-compressed), Archive (searchable). Error-driven learning with automatic pattern detection and fix attempts. Always active."
always: "true"
version: "1.0.0"
author: "lin82-2023"
metadata: >
  {"nanobot": {"always": true, "emoji": "🧠"}}
---

# Recall — Agent Memory & Auto-Improvement System

You have an intelligent memory system that not only records but learns and improves. Follow these rules strictly.

## Memory Locations

```
{workspace}/memory/
├── ESSENTIAL.md    # Core Memory — NEVER modify. Read-only.
├── MEMORY.md       # Working Memory — active long-term memory.
├── ERRORS.md       # Error Log — all failures with auto-fix attempts.
├── LEARNINGS.md    # Learning Rules — auto-generated from patterns.
├── PATTERNS.json   # Error statistics — internal use.
├── RECENT.md       # Dedup state — internal use.
├── HOOK_DEBUG.log  # Debug log — internal use.
└── archive/        # Archive — compressed cold memories.
```

## Session Start Protocol (MANDATORY)

**At the very beginning of EVERY conversation, read these 4 files:**

1. `memory/ESSENTIAL.md` — critical facts never forget
2. `memory/MEMORY.md` — current working memory  
3. `memory/ERRORS.md` — check for known errors in task area
4. `memory/LEARNINGS.md` — apply learned rules to avoid repeats

**Then check:**
5. `memory/PATTERNS.json` — if any high-frequency errors match current task

## Error Handling — The Improvement Loop

### Step 1: Detect (Automatic via Hook)
The system automatically detects:
- `exec` failures (non-zero exit code)
- `read_file`/`write_file`/`edit_file`/`list_dir` failures
- Python/Syntax errors in output

### Step 2: Analyze & Auto-Record
When an error is detected, the hook automatically:
1. **Classifies** the error type (PATH_ERROR, PERMISSION_ERROR, etc.)
2. **Records** to ERRORS.md with timestamp and context
3. **Updates** PATTERNS.json frequency count
4. **Generates** LEARNINGS.md rule after 2+ occurrences

### Step 3: Learn from Patterns
When an error type occurs **2+ times**, a learning rule is created:

```markdown
## PATH_ERROR (3 occurrences)
**Rule**: Before accessing any file or path, always check existence with test -e, ls -la, or read_dir; never assume a file or directory exists
**Apply**: Always
```

**You MUST apply these rules** in future similar tasks.

### Error Categories

| Category | Detection Pattern | Prevention Rule |
|----------|-------------------|-----------------|
| PATH_ERROR | "No such file"/"does not exist" | Check existence before access |
| PERMISSION_ERROR | "Permission denied" | Verify permissions, use sudo if needed |
| MISSING_COMMAND | "command not found" | Check with `which`, install if missing |
| NETWORK_ERROR | "connection"/"timeout"/"refused" | Add retry with backoff |
| PYTHON_ERROR | "ModuleNotFoundError"/"ImportError" | Verify dependencies before import |
| SYNTAX_ERROR | "SyntaxError" | Validate syntax before execution |
| FS_WRITE_ERROR | "failed to write"/"cannot create" | Check parent dir, disk space |
| FS_EDIT_ERROR | "old_text not found" | Read file first, use smaller unique snippets |
| FS_OFFSET_ERROR | "offset.*beyond end" | Adjust offset to file length |
| FS_READ_ERROR | "cannot read binary" | Skip binary files or use binary mode |
| UNKNOWN_ERROR | Others | Inspect closely, check logs |

## Manual Error Recording

If you detect an error that wasn't auto-recorded, manually add to ERRORS.md:

```markdown
## [ERR-YYYYMMDD-HHMMSS] ERROR_CATEGORY
**Logged**: YYYY-MM-DD HH:MM
**Status**: pending
**Category**: ERROR_CATEGORY

### What Failed
Command or operation that failed

### Error
Full error message

### Root Cause
Why it failed (optional)

### Fix Applied
What was done to fix (optional)
```

## Learning Rule Application (MANDATORY)

When LEARNINGS.md contains rules, you MUST apply them proactively:

**Before Learning:**
```
exec cat /tmp/some_file.txt
```

**After Learning (PATH_ERROR rule exists):**
```
exec test -e /tmp/some_file.txt && cat /tmp/some_file.txt || echo "File not found"
```

## Memory Consolidation

When MEMORY.md exceeds 4KB:
1. Old/referenced content moves to `archive/YYYY-MM-DD.md`
2. Key decisions stay in MEMORY.md
3. ESSENTIAL.md is NEVER compressed

## Your Goal

Become an agent that:
1. **Never makes the same mistake twice**
2. **Proactively prevents known error patterns**
3. **Continuously improves from every failure**
4. **Shares learnings across all sessions**

---

*Remember: Every error is a learning opportunity. Record it, fix it, prevent it.*

# Recall Skill

[![GitHub stars](https://img.shields.io/github/stars/lin82-2023/recall-skill?style=social)](https://github.com/lin82-2023/recall-skill/stargazers)

**Three-layer memory with automatic error learning and improvement**

## Features

- **🧠 Three-layer memory**: Core (essential) → Working (auto-compressed) → Archive (searchable)
- **🔍 Auto error detection**: Captures exec/filesystem failures automatically via hook
- **📈 Pattern learning**: Generates prevention rules after 2+ occurrences
- **🔄 Deduplication**: Prevents duplicate error logging within 60 seconds
- **📝 Full audit trail**: ERRORS.md, PATTERNS.json, LEARNINGS.md

## Installation

### Step 1: Copy skill to nanobot

```bash
mkdir -p /home/ubuntu/.nanobot/workspace/skills/recall
cp SKILL.md /home/ubuntu/.nanobot/workspace/skills/recall/
cp -r assets /home/ubuntu/.nanobot/workspace/skills/recall/
```

### Step 2: Initialize memory files

```bash
mkdir -p /home/ubuntu/.nanobot/workspace/memory/archive
cp assets/* /home/ubuntu/.nanobot/workspace/memory/
```

### Step 3: Inject hook (required!)

See **HOOK_INJECT.md** for detailed instructions. The hook must be injected into nanobot's `loop.py` for automatic error detection.

### Step 4: Restart nanobot

```bash
sudo systemctl restart nanobot
```

## How It Works

```
┌─────────────────────────────────────────────────────────┐
│                    Session Start                         │
│  Read: ESSENTIAL.md → MEMORY.md → ERRORS.md → LEARNINGS │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    Task Execution                        │
│  Agent performs actions with tools                      │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                 after_iteration hook                     │
│  1. Check tool_results for errors                       │
│  2. Classify error type                                 │
│  3. Record to ERRORS.md                                 │
│  4. Update PATTERNS.json                                │
│  5. Generate LEARNINGS.md rule (if 2+ occurrences)      │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                  Next Session                            │
│  Agent reads LEARNINGS.md and proactively avoids errors │
└─────────────────────────────────────────────────────────┘
```

## Error Categories

| Category | Detection | Prevention |
|----------|-----------|------------|
| PATH_ERROR | "No such file" | Check existence before access |
| PERMISSION_ERROR | "Permission denied" | Use sudo or check ownership |
| MISSING_COMMAND | "command not found" | Verify with `which` first |
| NETWORK_ERROR | "timeout"/"refused" | Add retry logic |
| PYTHON_ERROR | "ModuleNotFoundError" | Verify imports before running |
| SYNTAX_ERROR | "SyntaxError" | Validate before execution |
| FS_*_ERROR | File operation failures | Various checks |

## Files

```
recall/
├── SKILL.md           # Main skill instructions
├── HOOK_INJECT.md     # Hook injection guide
├── README.md          # This file
└── assets/
    ├── ESSENTIAL.md   # Core memory template
    ├── MEMORY.md      # Working memory template
    ├── ERRORS.md      # Error log template
    ├── LEARNINGS.md   # Learning rules template
    └── PATTERNS.json  # Error statistics template
```

## Testing

After installation, test by triggering errors:

```
User: 读取一个不存在的文件：/tmp/no_such_file_xyz.txt
```

Then check:

```bash
cat /home/ubuntu/.nanobot/workspace/memory/ERRORS.md
cat /home/ubuntu/.nanobot/workspace/memory/PATTERNS.json
cat /home/ubuntu/.nanobot/workspace/memory/HOOK_DEBUG.log
```

## License

MIT

## Author

[lin82-2023](https://github.com/lin82-2023)

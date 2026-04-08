# LEARNINGS.md — Learning Rules

This file contains rules auto-generated from error patterns. Apply these proactively.

## Generation Rules

- Rule created after 2+ occurrences of same error type
- Rules are specific and actionable
- Always check this file at session start

## Format

```markdown
## ERROR_CATEGORY (N occurrences)
**Rule**: Specific prevention action
**Apply**: Always
```

## Example Rules

### PATH_ERROR (3 occurrences)
**Rule**: Before accessing any file or path, always check existence with test -e, ls -la, or read_dir; never assume a file or directory exists
**Apply**: Always

### MISSING_COMMAND (2 occurrences)
**Rule**: Verify command exists with which <cmd> before using; if not found, install or use an alternative. NOTE: Always use python3, not python, on Linux
**Apply**: Always

# ERRORS.md — Error Log

This file is automatically populated by the recall hook. Do not edit manually.

## Format

```markdown
## [ERR-YYYYMMDD-HHMMSS] ERROR_CATEGORY
**Logged**: YYYY-MM-DD HH:MM
**Status**: pending
**Category**: PATH_ERROR

### What Failed
/path/to/file or command that failed

### Error
Full error message

---
```

## Error Categories

- PATH_ERROR — File/directory not found
- PERMISSION_ERROR — Access denied
- MISSING_COMMAND — Command not installed
- NETWORK_ERROR — Connection/timeout issues
- PYTHON_ERROR — Module import errors
- SYNTAX_ERROR — Code syntax errors
- FS_WRITE_ERROR — Write/create failures
- FS_EDIT_ERROR — Edit pattern not found
- FS_OFFSET_ERROR — Read offset beyond file
- FS_READ_ERROR — Binary file read as text
- UNKNOWN_ERROR — Unclassified errors

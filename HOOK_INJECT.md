# Hook Injection Guide

The recall skill requires a hook to be injected into nanobot's `loop.py` for automatic error detection.

## Prerequisites

- nanobot installed with `AgentLoop` class
- Python 3.10+
- Write access to nanobot's `agent/loop.py`

## Injection Steps

### 1. Locate loop.py

```bash
# Usually at:
NANOBOT_ENV=/home/ubuntu/nanobot-env
LOOP_PATH=$NANOBOT_ENV/lib/python3.12/site-packages/nanobot/agent/loop.py
```

### 2. Find _LoopHook class

The `_LoopHook` class is nested inside `AgentLoop._run_agent_loop`. Find the line with:

```python
class _LoopHook(AgentHook):
```

### 3. Add after_iteration method

Before the `def finalize_content` method in `_LoopHook`, add:

```python
            async def after_iteration(self, context: AgentHookContext) -> None:
                # ===== Recall Skill Enhanced Hook =====
                try:
                    import os as _os, datetime as _dt, json as _json, re as _re
                    from pathlib import Path as _Path
                    _md = _Path(str(loop_self.workspace)) / "memory"
                    _ep = _md / "ERRORS.md"
                    _pp = _md / "PATTERNS.json"
                    _lp = _md / "LEARNINGS.md"
                    _rp = _md / "RECENT.md"
                    _dp = _md / "HOOK_DEBUG.log"
                    _md.mkdir(parents=True, exist_ok=True)
                    _patterns = {}
                    if _pp.exists():
                        try:
                            with open(_pp, "r") as _f:
                                _patterns = _json.load(_f)
                        except Exception:
                            pass

                    _fs_tools = {"read_file", "write_file", "edit_file", "list_dir"}
                    _epfx = "Error:"

                    _cat_rules = {
                        "PATH_ERROR": "Before accessing any file or path, always check existence with test -e, ls -la, or read_dir; never assume a file or directory exists",
                        "PERMISSION_ERROR": "Check user with whoami; try sudo when permission denied; verify file/dir permissions with ls -la",
                        "MISSING_COMMAND": "Verify command exists with which <cmd> before using; if not found, install or use an alternative. NOTE: Always use python3, not python, on Linux",
                        "NETWORK_ERROR": "Add retry logic with exponential backoff for network operations; check connectivity first",
                        "FS_WRITE_ERROR": "Before write_file/edit_file, verify parent directory exists with list_dir; check disk space; ensure UTF-8 encoding",
                        "FS_EDIT_ERROR": "Before edit_file, always read_file first to verify old_text exists EXACTLY (including whitespace); use smaller unique snippets",
                        "PYTHON_ERROR": "Run python3 -c 'import <module>' to verify Python dependencies; install with pip3 install before importing",
                        "SYNTAX_ERROR": "Check syntax with python3 -m py_compile before running; validate JSON/YAML with jq/yq first",
                        "FS_OFFSET_ERROR": "When offset exceeds file lines, adjust offset to last line or remove offset parameter",
                        "FS_READ_ERROR": "Binary files cannot be read as text; skip or use appropriate binary handler",
                        "UNKNOWN_ERROR": "Inspect error message closely; try with verbose/debug flags; check logs for root cause",
                    }

                    def _detect(_rs, _name):
                        _ec = None
                        _err_lc = _rs.lower()
                        for _line in _rs.split("\n"):
                            if _line.strip().startswith("Exit code:"):
                                try:
                                    _ec = int(_line.strip().split(":")[1].strip())
                                except:
                                    pass
                        if _name == "exec" and _ec is not None and _ec != 0:
                            if any(x in _rs for x in ["No such file", "ENOENT", "does not exist"]):
                                return "PATH_ERROR"
                            elif "Permission denied" in _rs:
                                return "PERMISSION_ERROR"
                            elif any(x in _err_lc for x in ["command not found", "not found:", ": not found"]):
                                return "MISSING_COMMAND"
                            elif any(x in _err_lc for x in ["connection", "timeout", "refused", "network", "dns"]):
                                return "NETWORK_ERROR"
                            elif any(x in _err_lc for x in ["traceback", "error:", "exception"]):
                                if "syntaxerror" in _err_lc:
                                    return "SYNTAX_ERROR"
                                elif any(x in _err_lc for x in ["modulenotfounderror", "importerror", "no module named", "import "]):
                                    return "PYTHON_ERROR"
                                return "PYTHON_ERROR"
                            return "UNKNOWN_ERROR"
                        elif _name in _fs_tools and _rs.startswith(_epfx):
                            if any(x in _err_lc for x in ["not found", "no such file", "does not exist", "unknown path"]):
                                return "PATH_ERROR"
                            elif "permission denied" in _err_lc:
                                return "PERMISSION_ERROR"
                            elif any(x in _err_lc for x in ["failed to write", "cannot write", "failed to create", "disk full", "no space"]):
                                return "FS_WRITE_ERROR"
                            elif any(x in _err_lc for x in ["old_text not found", "best match", "no match", "edit failed"]):
                                return "FS_EDIT_ERROR"
                            elif "offset" in _err_lc and "beyond end" in _err_lc:
                                return "FS_OFFSET_ERROR"
                            elif "cannot read binary" in _err_lc:
                                return "FS_READ_ERROR"
                            return "UNKNOWN_ERROR"
                        return None

                    def _extract_path(_args):
                        if isinstance(_args, dict):
                            for _k in ["path", "file_path", "file", "command", "target", "url"]:
                                if _k in _args:
                                    return str(_args[_k])
                            return str(_args)
                        elif isinstance(_args, list) and _args:
                            _a = _args[0]
                            if isinstance(_a, dict):
                                for _k in ["path", "file_path", "file", "command", "target", "url"]:
                                    if _k in _a:
                                        return str(_a[_k])
                                return str(_a)
                            return str(_a)
                        return str(_args)

                    _now_s = _dt.datetime.now().strftime("%Y%m%d-%H%M%S")
                    _now_h = _dt.datetime.now().strftime("%Y-%m-%d %H:%M")
                    _new_errors = []
                    _tcs = list(context.tool_calls)
                    _trs = list(context.tool_results)

                    for _i, _tc in enumerate(_tcs):
                        _rs = str(_trs[_i] if _i < len(_trs) else "")
                        _cmd = _extract_path(_tc.arguments)
                        _cat = _detect(_rs, _tc.name)
                        if not _cat:
                            continue
                        if _cat not in _patterns:
                            _patterns[_cat] = {"count": 0, "examples": []}
                        _patterns[_cat]["count"] += 1
                        _sig = _re.sub(r"\s+", " ", _rs[:100]).strip()
                        _dup = False
                        if _rp.exists():
                            try:
                                _recent = _json.load(open(_rp))
                                if _dt.datetime.now().timestamp() - _recent.get("ts", 0) < 60 and _recent.get("sig") == _sig:
                                    _dup = True
                            except:
                                pass
                        if not _dup:
                            with open(_rp, "w") as _rf:
                                _json.dump({"ts": _dt.datetime.now().timestamp(), "sig": _sig}, _rf)
                            _entry = "\n".join(["", "## [ERR-" + _now_s + "] " + _cat,
                                "**Logged**: " + _now_h, "**Status**: pending", "**Category**: " + _cat,
                                "", "### What Failed", _cmd[:500], "", "### Error", _rs[:800], "", "---", ""])
                            with open(_ep, "a") as _ef:
                                _ef.write(_entry)
                            _new_errors.append(_cat)
                            with open(_pp, "w") as _f:
                                _json.dump(_patterns, _f, indent=2)
                        if _patterns[_cat]["count"] >= 2 and _cat in _cat_rules:
                            _already = False
                            if _lp.exists():
                                try:
                                    if _cat in open(_lp).read():
                                        _already = True
                                except:
                                    pass
                            if not _already:
                                _lr = "\n".join(["", "## " + _cat + " (" + str(_patterns[_cat]["count"]) + " occurrences)",
                                    "**Rule**: " + _cat_rules[_cat], "**Apply**: Always", "", "---", ""])
                                with open(_lp, "a") as _lf:
                                    _lf.write(_lr)
                    with open(_dp, "a") as _df:
                        _df.write("[" + _now_h + "] tools=" + str([_tc.name for _tc in _tcs]) + " detected=" + str(_new_errors) + "\n")
                except Exception as _ex:
                    import traceback
                    with open("/tmp/recall_hook_exc.log", "a") as _ef:
                        _ef.write("[" + _dt.datetime.now().strftime("%Y-%m-%d %H:%M:%S") + "] EXC: " + str(_ex) + "\n")
                        _ef.write(traceback.format_exc() + "\n")
                # ===== End Recall Skill Hook =====

```

**Important**: The indentation is 12 spaces (inside `_LoopHook` class, which is inside `_run_agent_loop` method).

### 4. Restart nanobot

```bash
sudo systemctl restart nanobot
```

### 5. Verify

Check that errors are being logged:

```bash
# After triggering an error, check:
cat /home/ubuntu/.nanobot/workspace/memory/ERRORS.md
cat /home/ubuntu/.nanobot/workspace/memory/HOOK_DEBUG.log
```

## Troubleshooting

### Hook not firing

1. Check syntax: `python3 -m py_compile $LOOP_PATH`
2. Check exception log: `cat /tmp/recall_hook_exc.log`
3. Verify `after_iteration` is in `_LoopHook` class

### Errors not recorded

1. Check HOOK_DEBUG.log for entries
2. Verify memory directory exists: `ls -la /home/ubuntu/.nanobot/workspace/memory/`
3. Check PATTERNS.json is writable

### ImportError on startup

The hook code uses only standard library modules, so no additional packages needed.

## Notes

- The hook runs after every iteration, adding minimal overhead
- Deduplication prevents logging same error within 60 seconds
- LEARNINGS rules are generated after 2+ occurrences
- The hook is scoped with `_` prefix to avoid conflicts

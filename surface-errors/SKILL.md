---
name: surface-errors
description: 'Use this skill whenever writing or editing Python or Bash code, or any command whose behavior on failure isn''t fully known. Enforces letting errors surface instead of silently swallowing them. Trigger this for ANY script-writing or script-editing task, not just ones that explicitly mention error handling — code often has at least one place where a failure could be hidden (an API call, a subprocess, a file operation, a loop over unreliable data). Also apply it when reviewing or debugging existing code that hides errors, e.g. bare except, except Exception, except-pass, set +e, redirecting stderr to /dev/null, or-true fallback, or ignored/unchecked return codes.'
---

# Surface Errors, Don't Hide Them

## Core principle

When calling a function, command, or API that you (Claude) are not fully certain about — you haven't read its docstring/source, you're guessing at arguments, or you don't know its exact failure modes — **do not wrap it in a broad try/except or redirect its errors into a black hole.** Let it fail loudly. A crash with a full traceback is easy to debug. A silently-swallowed exception is a bug that surfaces days later somewhere completely unrelated, with no trace of its origin.

This applies broadly, not just to "unfamiliar" calls: default to precise, narrow, or absent exception handling. Only catch a specific exception type when there's genuinely no way to prevent it from being raised in the first place (e.g. a network call that can always time out) — and even then, catch the *specific* exception, not `Exception` or bare `except:`.

If you (Claude) find yourself reaching for a broad `except Exception`, `except:`, or `2>/dev/null 2>&1` because you're not sure what a function/command can raise or how it can fail — that uncertainty is the signal to go read the docs/source/`--help` first, not to paper over it.

## The family of patterns to avoid

These all have the same failure mode in common: they turn a debuggable, loud failure into a silent, hard-to-trace one.

**Python:**
- `except Exception:` or bare `except:` — catches everything, including bugs you didn't anticipate (KeyboardInterrupt, TypeErrors from your own mistakes, etc.)
- `except: pass` / `except Exception: continue` / `except Exception: return None` — catches AND discards, so there isn't even a log line
- A giant `try:` block wrapping many unrelated statements, so you can't tell which one actually failed
- `subprocess.run(..., check=False)` (or omitting `check=True`) when you're not deliberately handling the failure case afterward

**Bash:**
- `command 2>/dev/null`, `command >/dev/null 2>&1` — throws away stderr/stdout so you can't see what went wrong
- `command || true` / `command; true` — forces success exit code regardless of outcome
- `set +e` (disabling bash's fail-fast behavior) without a very deliberate, narrow reason.
- Not checking `$?` after a command whose success actually matters

**General/any language:** the same principle holds — a catch-all error handler with no re-raise, no log, and no specific exception type is a red flag regardless of the syntax.

## What to do instead

1. **If you're unsure how a function/command fails:** look it up (docstring, `--help`, source, man page) before writing the call, rather than guessing and wrapping it defensively.
2. **If failure is expected and you genuinely can't prevent/anticipate it** (flaky network, external API, optional file that may not exist): catch the *specific* exception type, and either handle it meaningfully (retry, fallback with a logged reason) or re-raise with added context. Never silently discard it.
3. **In Bash:** start scripts with `set -eu` (or with `set -euo pipefail` if it is not a POSIX `/bin/sh` script), unless there's a specific reason not to, let commands fail loudly by default.
4. **If you deliberately do want to suppress or ignore a failure** (rare, and should be a conscious choice, not a default): say so explicitly — add a comment explaining *why* it's safe to ignore this particular failure — rather than silently swallowing it as a matter of habit.
5. **Don't preemptively defend against errors you haven't seen yet.** Write the straightforward version first, let it fail if it fails. Most failures can be anticipated by checking preconditions up front, rather than by wrapping the call in `try/except`.

## Examples

### Python

Avoid:
```python
try:
    data = fetch_user(user_id)
except Exception:
    continue
```

Prefer (if you don't yet know how `fetch_user` fails):
```python
data = fetch_user(user_id)  # let it raise; investigate the traceback if it does
```

Prefer (if you've confirmed it raises `UserNotFoundError` and that's an expected, handleable case):
```python
try:
    data = fetch_user(user_id)
except UserNotFoundError:
    logger.warning(f"No user found for id={user_id}, skipping")
    continue
```

### Bash

Avoid:
```bash
some_tool --guessed-flag=value >/dev/null 2>&1
```

Prefer:
```bash
set -eu # Use `set -euo pipefail` if for sure we are not in a POSIX `/bin/sh` file
some_tool --guessed-flag=value
```
(and if the flag turns out to be wrong, you'll see the real error immediately instead of a silent no-op)

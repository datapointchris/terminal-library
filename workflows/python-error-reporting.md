---
tags: [python, errors, exceptions, logging, traceback, alerting]
---

# python error reporting — str vs repr vs type, and which channel gets the traceback

An alert that goes out saying `error: ` with nothing after it is the normal
outcome of interpolating `{e}`. Pick the spelling from what the reader needs.

## What each spelling actually gives you

```text
type              str(e)              repr(e)  ==  f'{e!r}'
ValueError        'bad thing'         ValueError('bad thing')
KeyError          "'batch_id'"        KeyError('batch_id')      <- str() ADDS quotes
IndexError        ''                  IndexError()              <- str() is EMPTY
Bare()            ''                  Bare()                    <- str() is EMPTY
RuntimeError('')  ''                  RuntimeError('')          <- str() is EMPTY
```

`repr(e)` already contains the class name, so pairing it with
`type(e).__name__` prints the type twice.

## For humans, the stdlib already has it

```python
import traceback

def describe_exception(error: BaseException) -> str:
    return ''.join(traceback.format_exception_only(error)).rstrip()

# ValueError('bad thing')       -> 'ValueError: bad thing'
# TrackingTimeout()             -> 'tracking.TrackingTimeout'   <- module-qualified
# with e.add_note('batch b7f2') -> 'ValueError: bad thing\nbatch b7f2'
```

It returns a **list** and each element ends in `\n`, hence the join and rstrip.
Builtins stay bare; anything else is qualified by module. Beats hand-rolling
`f'{type(e).__name__}: {e}'`, which drops what the next block shows.

## What str(e) silently drops

```python
e = ValueError('bad thing'); e.add_note('batch b7f2c1')   # PEP 678, 3.11+
f'{type(e).__name__}: {e}'          # 'ValueError: bad thing'          <- note GONE
describe_exception(e)               # 'ValueError: bad thing\nbatch b7f2c1'

# SyntaxError: str() flattens to a parenthetical, stdlib keeps the source
f'{type(se).__name__}: {se}'        # 'SyntaxError: bad syntax (f.py, line 3)'
describe_exception(se)              # '  File "f.py", line 3\n    x = = 1\n        ^^^
                                    #  SyntaxError: bad syntax'
```

## error() vs exception()

```python
logger.error(msg)                   # message only
logger.exception(msg)               # message + traceback; only valid inside `except`
logger.error(msg, exc_info=True)    # same as exception(), usable anywhere

except MyTimeout:                   # expected, raised by your own control flow
    logger.error(msg)               # its traceback points at your code obeying you

except Exception:                   # unexpected — nobody knows what happened
    logger.exception(msg)           # the traceback IS the diagnostic

''.join(traceback.format_exception(e))   # full stack as a string, if you need it raw
```

## Which channel gets what

```text
logs                type + message + FULL TRACEBACK, via logger.exception
                    the only place the stack belongs

human (email/chat)  describe_exception(e), plus a searchable id that appears in
                    the log lines. never the traceback — unreadable, and it
                    leaks file paths and internals to whoever is on the list

machine (API/queue) type(e).__name__ in the NAME field — bare and stable, so a
                    matcher can compare against it. describe_exception(e) in
                    the DETAIL field. both inside that API's documented limits

always              re-raise after reporting, so the runtime records it too
```

AWS Step Functions `send_task_failure`, from botocore's service model: `error`
max 256 and is what `Catch`/`Retry` match on with `ErrorEquals`; `cause` max
32768. `str(e)` over 256 chars in `error` raises ValidationException from
inside the except block, and that becomes the reported failure instead of the
real one. `describe_exception` is wrong here too — it qualifies by module, so
`ErrorEquals: [TrackingTimeout]` would never match `tracking.TrackingTimeout`.

## `__name__` is not a dunder method

The "never call dunders" rule is about **protocol hooks** — `len(x)` not
`x.__len__()` — where the builtin does validation the hook does not. `__name__`
is a plain `str` attribute with no builtin accessor, and CPython's own 3.13
stdlib reads `type(x).__name__` 106 times across 61 files.

```python
callable(ValueError.__name__)   # False — it is data
type((1).__index__)             # <class 'method-wrapper'> — that one is a hook
```

There is no `e.name` because inside Python you never need the name as a string;
you catch and `isinstance` by the type object itself. You only need the text at
a boundary where you leave the language.

## Gotchas

```python
# The type, twice. repr already carries it.
cause = f'{type(e).__name__}: {e!r}'     # 'ValueError: ValueError('bad thing')'

# A report that fails inside an except block BECOMES the reported error.
try:
    send_alert(body)
except Exception:
    logger.exception('alert failed; original error stands')   # swallow the send
# original exception keeps propagating — never let the alert replace the cause

# `finally` is the same trap and it is worse: it fires on the SUCCESS path too,
# so an unguarded cleanup there can invent a failure where there was none.
finally:
    try:
        write_final_state()
    except Exception:
        logger.exception('cleanup failed; original outcome stands')
```

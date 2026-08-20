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

`str(e)` is empty for `raise SomeError()` with no argument, and for any library
that carries its detail on an attribute instead. `repr(e)` already contains the
class name, so pairing it with `type(e).__name__` prints the type twice.

## The formatter that cannot render blank

```python
def describe_exception(error: BaseException) -> str:
    message = str(error)
    return f'{type(error).__name__}: {message}' if message else type(error).__name__

# ValueError('bad thing') -> 'ValueError: bad thing'
# Bare()                  -> 'Bare'
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

import traceback
''.join(traceback.format_exception(e))   # full stack, as a string
traceback.format_exception_only(e)       # just `Type: message`, no frames
```

## Which channel gets what

```text
logs                type + message + FULL TRACEBACK, via logger.exception
                    the only place the stack belongs

human (email/chat)  type + message, plus a searchable id that appears in the
                    log lines. never the traceback — unreadable, and it leaks
                    file paths and internals to whoever is on the list

machine (API/queue) type in the NAME field, formatted message in the DETAIL
                    field, both inside that API's documented limits

always              re-raise after reporting, so the runtime records it too
```

AWS Step Functions `send_task_failure`, from botocore's service model:
`error` max 256 and is what `Catch`/`Retry` match on with `ErrorEquals`, so it
takes `type(e).__name__`; `cause` max 32768 and takes the message. A `str(e)`
over 256 chars in `error` raises ValidationException from inside the except
block, and that becomes the reported failure instead of the real one.

## Gotchas

```python
# The type, twice. repr already carries it.
cause = f'{type(e).__name__}: {e!r}'     # 'ValueError: ValueError('bad thing')'
cause = describe_exception(e)            # 'ValueError: bad thing'

# A report that fails inside an except block BECOMES the reported error.
try:
    send_alert(body)
except Exception:
    logger.exception('alert failed; original error stands')   # swallow the send
# original exception keeps propagating — never let the alert replace the cause
```

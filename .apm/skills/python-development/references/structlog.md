# structlog — JSON by default, console for localhost

The rule: **structured JSON logs in every real environment, human-readable
console logs only on localhost/dev**, with the renderer chosen from the
environment rather than edited by hand.

Both renderers carry the same fields: a `message` (structlog's `event` renamed)
and callsite info (`filename`, `lineno`, `func_name`) on every line — see
[What each log line contains](#what-each-log-line-contains).

## Deciding the renderer

Pick console rendering when running locally. Two common signals — use either or
both:

- An explicit env var: `ENV` / `APP_ENV` is `local` or `dev`.
- A TTY attached to stderr (`sys.stderr.isatty()`), i.e. an interactive shell.

Default to JSON whenever neither says "local".

## `configure_logging()`

```python
import logging
import os
import sys

import structlog


def _use_console_renderer() -> bool:
    env = os.getenv("APP_ENV", os.getenv("ENV", "")).lower()
    if env in {"local", "dev", "development"}:
        return True
    if env:  # any other explicit environment => JSON
        return False
    return sys.stderr.isatty()  # fall back to "interactive shell => console"


def configure_logging(level: str = "INFO") -> None:
    """Configure structlog. Call once at startup, before anything logs."""
    console = _use_console_renderer()

    shared_processors: list[structlog.typing.Processor] = [
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.StackInfoRenderer(),
        # Callsite info on every line: which file/line/function emitted it.
        structlog.processors.CallsiteParameterAdder(
            {
                structlog.processors.CallsiteParameter.FILENAME,
                structlog.processors.CallsiteParameter.LINENO,
                structlog.processors.CallsiteParameter.FUNC_NAME,
            }
        ),
        structlog.processors.TimeStamper(fmt="iso", utc=True),
    ]

    # Render the log message under `message` (not structlog's default `event`),
    # consistently in both console and JSON. ConsoleRenderer is told the new key
    # so it still prints the message positionally.
    renderer: structlog.typing.Processor = (
        structlog.dev.ConsoleRenderer(event_key="message")
        if console
        else structlog.processors.JSONRenderer()
    )

    structlog.configure(
        processors=[
            *shared_processors,
            # Exceptions: readable tracebacks locally, structured dict for JSON.
            structlog.processors.format_exc_info
            if console
            else structlog.processors.dict_tracebacks,
            # Rename `event` -> `message`; must run last, just before the renderer.
            structlog.processors.EventRenamer("message"),
            renderer,
        ],
        wrapper_class=structlog.make_filtering_bound_logger(
            logging.getLevelName(level)
        ),
        logger_factory=structlog.PrintLoggerFactory(),
        cache_logger_on_first_use=True,
    )
```

Call it once at process start:

```python
configure_logging(level=os.getenv("LOG_LEVEL", "INFO"))
log = structlog.get_logger()
```

## What each log line contains

Every line — console and JSON — carries the same standard keys:

| Key | Where it comes from |
|-----|---------------------|
| `timestamp` | `TimeStamper` (ISO 8601, UTC) |
| `level` | `add_log_level` |
| `message` | the event, renamed from `event` by `EventRenamer("message")` |
| `filename`, `lineno`, `func_name` | `CallsiteParameterAdder` — the call site |
| bound context + event key/values | `merge_contextvars` and the `key=value`s you pass |

Console output is structlog's standard
[`ConsoleRenderer`](https://www.structlog.org/en/stable/console-output.html) look
with the callsite fields appended; JSON is one object per line with the same keys
(`"message"`, never `"event"`).

**Call sites don't change.** The first positional argument is still the event
*name* — `log.info("user.created", user_id=uid)`. `EventRenamer` only renames the
rendered key; you never pass `message=` yourself.

## Using the logger

```python
log = structlog.get_logger()

# Event name + key/values — never f-string data into the message.
log.info("user.created", user_id=user.id, plan=user.plan)

# Bind context that should ride along with every later log in this scope.
structlog.contextvars.bind_contextvars(request_id=request_id)
try:
    ...
finally:
    structlog.contextvars.clear_contextvars()
```

## Routing stdlib logging through structlog (optional)

Third-party libraries use stdlib `logging`. To render their records through the
same pipeline, add a `ProcessorFormatter` handler on the root logger and feed
records into the shared processor chain. Do this when you need library logs to
match the app's JSON/console format; skip it for simple services.

See the structlog docs for the `ProcessorFormatter` "standard library" recipe.

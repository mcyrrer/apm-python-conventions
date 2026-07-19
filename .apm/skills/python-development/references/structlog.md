# structlog — JSON by default, console for localhost

The rule: **structured JSON logs in every real environment, human-readable
console logs only on localhost/dev**, with the renderer chosen from the
environment rather than edited by hand.

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
        structlog.processors.TimeStamper(fmt="iso", utc=True),
    ]

    renderer: structlog.typing.Processor = (
        structlog.dev.ConsoleRenderer()
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
            renderer,
        ],
        wrapper_class=structlog.make_filter_bound_logger(
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

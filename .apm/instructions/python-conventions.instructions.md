---
description: Always-on Python conventions — structlog logging, FastAPI, FastMCP, typing.
applyTo: "**/*.py"
---

# Python conventions

Default conventions for Python code in this project. The full setup recipes and
copy-paste snippets live in the `python-development` skill; load it when
scaffolding a project or wiring one of these tools.

## Logging

- **Use `structlog`** for all logging — never `print` and never bare stdlib
  `logging` for application logs.
- **JSON by default, console for localhost.** Emit JSON logs in every real
  environment; use the human-readable console renderer only in local/dev. Select
  the renderer from the environment, not by hand-editing before each run.
- **Structured, not interpolated.** Log an event name plus key/value pairs
  (`log.info("user.created", user_id=uid)`), don't f-string data into the
  message. Bind request/task context once with `log.bind(...)` / `contextvars`.
- **Standard keys on every line.** Each log line carries the message plus callsite
  fields (`filename`, `lineno`, `func_name`) in both console and JSON. The message
  is rendered under the `message` key (structlog's default `event`, renamed) — the
  positional event name at the call site is unchanged; only the output key differs.

## HTTP APIs

- **FastAPI is the default** web framework. Prefer `async def` endpoints.
- Use **Pydantic** models for request/response bodies and `pydantic-settings`
  for configuration.
- Inject shared resources (DB session, settings, clients) via `Depends`, not
  module-level globals.

## MCP servers

- **Use FastMCP** (`from fastmcp import FastMCP`) to build MCP servers; define
  tools/resources with its decorators rather than hand-rolling a server.

## Core conventions

- **Type hints** on every public function signature; keep code type-checker
  clean.
- **Prefer `async`** when the surrounding framework is async (FastAPI, FastMCP,
  async SQLAlchemy) — don't block the event loop with sync I/O.
- **Standard layout:** a `src/`-style importable package, dependencies declared
  and pinned in `pyproject.toml`.
- **Clean, PEP 8-compliant style** is expected on every file — see
  `clean-code.instructions.md` for the rules and the `code-quality` skill for the
  Ruff + mypy setup that enforces them.
- Data-layer work (SQLAlchemy / Alembic / PostgreSQL) follows
  `database.instructions.md` and the `database-migrations` skill.

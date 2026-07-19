---
name: python-development
description: >-
  Use when setting up or scaffolding a Python project, wiring up logging, or
  building a FastAPI HTTP service or a FastMCP server. Applies this project's
  defaults — structlog (JSON in production, console for localhost), FastAPI for
  HTTP APIs, FastMCP for MCP servers, typed and async-first — with copy-paste
  config. For the data layer (SQLAlchemy/Alembic) use the database-migrations skill.
---

# Python development

Set up Python services the way this project standardizes them. Keep the always-on
rules in `python-conventions.instructions.md` in mind; this skill is the deeper
"how", with runnable snippets in `references/`.

## Defaults at a glance

- **Logging:** `structlog` — JSON renderer in every real environment, console
  renderer only for localhost/dev, chosen from the environment. Every line
  carries callsite fields (`filename`, `lineno`, `func_name`) and the message
  under the `message` key.
- **HTTP API:** FastAPI, `async def` endpoints, Pydantic models, config via
  `pydantic-settings`, shared resources injected with `Depends`.
- **MCP server:** FastMCP (`from fastmcp import FastMCP`) with decorator-defined
  tools/resources.
- **Baseline:** type hints on public signatures, async-first, `src/`-style
  package, deps pinned in `pyproject.toml`.

## Setting up a service

1. **Project skeleton.** Create a `src/<pkg>/` importable package and a
   `pyproject.toml` with pinned dependencies (`structlog`, plus `fastapi` +
   `uvicorn` or `fastmcp` as needed).
2. **Configure logging first.** Add a `configure_logging()` and call it at
   startup, before anything logs. Use the env-driven JSON-vs-console switch from
   `references/structlog.md`.
3. **Build the app.**
   - HTTP API → follow `references/fastapi.md` (app factory, routers, settings,
     request-logging middleware).
   - MCP server → follow `references/fastmcp.md` (server skeleton, tools,
     transport).
4. **Wire logging into the framework** so requests/tool calls carry bound context
   (request id, tool name) — see the integration sections in the framework
   references.
5. If the service needs a database, continue with the `database-migrations` skill.

## References (load as needed)

- `references/structlog.md` — the `configure_logging()` recipe with the
  JSON-default / console-for-localhost switch, processor chain, callsite fields,
  and the `message` key.
- `references/fastapi.md` — FastAPI app factory, settings, and structlog
  request-logging middleware.
- `references/fastmcp.md` — FastMCP server skeleton, tools/resources, transports,
  and structlog integration.

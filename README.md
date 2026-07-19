# python-conventions

An **[APM](https://microsoft.github.io/apm/) producer package** that captures a
team's **Python backend conventions** as agent context: always-on instructions
plus load-on-demand setup skills, so any AI coding agent scaffolds and reviews
Python services the same way.

> [APM](https://microsoft.github.io/apm/producer/) (Agent Package Manager) is
> "a dependency manager for AI agents — like npm for agent context." This is a
> **producer** repo: it authors primitives (skills, instructions) that consumers
> install and `apm compile` deploys to Claude, Copilot, Cursor, and other agents.

## What's inside

```
.apm/
├── skills/
│   ├── python-development/       # structlog / FastAPI / FastMCP setup (+ references/)
│   └── database-migrations/      # SQLAlchemy / Alembic + migration-tests CI (+ references/, assets/)
└── instructions/
    ├── python-conventions.instructions.md   # always-on Python defaults
    └── database.instructions.md             # always-on DB / migrations defaults
```

| Primitive | What it gives your agent |
|-----------|--------------------------|
| **`python-conventions` instructions** | Always-on Python defaults (structlog, FastAPI, FastMCP, typing, async). |
| **`python-development` skill** | Setup recipes for structlog (JSON/console), FastAPI, and FastMCP, with copy-paste config. |
| **`database` instructions** | Always-on DB defaults (SQLAlchemy, Alembic-per-change, Postgres 17, CI-enforced migration tests). |
| **`database-migrations` skill** | SQLAlchemy + Alembic setup on PostgreSQL 17, and a ready-to-copy migration-tests CI workflow. |

## The conventions

Backend-Python defaults captured as always-on instructions plus load-on-demand
skills:

- **Logging** — `structlog`, JSON in production and the console renderer for
  localhost, selected from the environment (`python-development` skill).
- **Frameworks** — **FastAPI** for HTTP APIs and **FastMCP** for MCP servers,
  async-first and typed.
- **Data layer** — **SQLAlchemy** (typed 2.0 models) with **Alembic** for every
  schema change, defaulting to **PostgreSQL 17**. Adding Alembic **always**
  installs a migration-tests GitHub Actions workflow (`upgrade head`,
  `alembic check`, downgrade→upgrade roundtrip) so migrations can't silently
  drift or break (`database-migrations` skill).

## Install (consumer)

Add it to a consuming project's `apm.yml`:

```yaml
dependencies:
  apm:
    - mcyrrer/apm-python-conventions#v0.2.0
```

then `apm install` and `apm compile`. Or install directly:

```bash
apm install mcyrrer/apm-python-conventions
```

Once installed, the instructions apply automatically and the skills load when you
ask your agent to set up logging, a FastAPI/FastMCP service, or a database layer.

## Working on this repo

- [`CLAUDE.md`](CLAUDE.md) — how to work in this repository.
- [`docs/apm-format-reference.md`](docs/apm-format-reference.md) — granular APM
  format reference (exact frontmatter for every primitive) so changes stay
  in-spec.
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — contribution workflow.

## Releases

Versioning and tagging are automated. Run the **Version bump** workflow from the
Actions tab (`patch` / `minor` / `major`) to open a release PR; merging it tags
`v<version>` and publishes a GitHub Release. See
[`CONTRIBUTING.md`](CONTRIBUTING.md#releasing).

## License

MIT — see [`LICENSE`](LICENSE).

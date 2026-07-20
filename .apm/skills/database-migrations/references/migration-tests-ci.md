# Migration-tests CI

Every repo that uses Alembic must run a migration-tests workflow in CI. The
ready-to-copy file is `assets/migration-tests.yml`; drop it in at
`.github/workflows/migration-tests.yml` and adjust the `DATABASE_URL` driver and
the package/`env.py` import path. It installs deps with `uv sync` by default.

## What it asserts, and why

Against a **fresh PostgreSQL 17** service container:

1. **`alembic upgrade head` succeeds.** Proves every migration applies cleanly
   from an empty database — catches a migration that references a missing table,
   a bad dependency order, or SQL that errors on a real engine (things a purely
   offline diff won't surface).
2. **`alembic check` reports no changes.** Proves models and migrations are in
   sync — catches a model edit that shipped without a matching migration (schema
   drift), the single most common migration bug.
3. **Downgrade → upgrade roundtrip on the latest revision.** `downgrade -1` then
   `upgrade head` proves `downgrade()` actually reverses `upgrade()` and that the
   revision is reversible — catches empty/incorrect downgrades and irreversible
   operations before they reach `main`.

Running against real Postgres 17 (not SQLite) is deliberate: it exercises the
same engine, types, and constraints as production, so dialect-specific problems
fail in CI instead of at deploy time.

## Adapting the workflow

- **Install step:** defaults to `uv sync` (deps and venv managed by uv). Only
  change it if the project uses a different manager, so `alembic` and the app
  package stay importable.
- **`DATABASE_URL`:** the workflow exports one pointing at the service container;
  match the driver your `env.py`/app expects (`postgresql+psycopg://` for sync,
  `postgresql+asyncpg://` for async). The async Alembic template can run sync
  migrations against a sync URL in CI even if the app is async — keep them
  consistent with your `env.py`.
- **Migrations path:** if your Alembic directory isn't the default, pass
  `-c path/to/alembic.ini` to each command.
- **Postgres version:** stays `postgres:17` unless the project standardizes on
  another version.

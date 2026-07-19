# Alembic conventions

Alembic owns the schema. Every schema change ships as a reviewed migration.

## Initialize

```bash
alembic init -t async migrations   # use the async template with async SQLAlchemy
```

(Use the default template for a sync engine.)

## Wire `env.py`

Point Alembic at your models' metadata and the runtime database URL so
autogenerate and `alembic check` work:

```python
# migrations/env.py
import os

from myapp.db import Base          # the shared DeclarativeBase
from myapp import models           # noqa: F401 — import so all tables register

target_metadata = Base.metadata

config = context.config
config.set_main_option(
    "sqlalchemy.url",
    os.environ["DATABASE_URL"],    # never hard-code credentials in alembic.ini
)
```

Set `compare_type=True` (and `compare_server_default=True` if you rely on server
defaults) in the `context.configure(...)` calls so autogenerate detects column
type/default changes.

## Everyday workflow

```bash
# After changing models, generate a migration:
alembic revision --autogenerate -m "add users table"

# ALWAYS review the generated file before committing — confirm it captures the
# intended change and nothing spurious (dropped columns, reordered ops).

alembic upgrade head     # apply
alembic downgrade -1     # roll back one revision
alembic current          # show applied revision
alembic history          # list revisions
```

## `alembic check`

`alembic check` autogenerates against the current DB and **fails if there are
un-generated changes** — i.e. models drifted from migrations. Run it locally and
in CI so a model change without a matching migration can't merge:

```bash
alembic upgrade head
alembic check           # exits non-zero if models and migrations disagree
```

## Rules

- One logical change per migration; give it a descriptive message.
- Provide a real `downgrade()` — the CI roundtrip exercises it.
- Never edit an already-merged/applied migration; add a new one.
- Keep `DATABASE_URL` in the environment; default the DB to **PostgreSQL 17**.

The `database-migrations` skill's `assets/migration-tests.yml` runs these checks
in CI — see `references/migration-tests-ci.md`.

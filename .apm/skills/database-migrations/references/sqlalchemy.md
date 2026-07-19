# SQLAlchemy conventions

Typed, 2.0-style SQLAlchemy. One `Base`, one engine, one `sessionmaker`, sessions
handed to callers via a dependency.

## Base and typed models

```python
from datetime import datetime

from sqlalchemy import String, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(320), unique=True, index=True)
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())
```

Use `Mapped[...]` + `mapped_column` for every column so models are fully typed.
Alembic autogenerate reads `Base.metadata`, so keeping models on this one `Base`
is what lets migrations be generated and checked.

## Engine and session

Prefer async with FastAPI/FastMCP. `DATABASE_URL` uses the `postgresql+asyncpg://`
driver; default the database to **PostgreSQL 17**.

```python
from sqlalchemy.ext.asyncio import (
    AsyncSession,
    async_sessionmaker,
    create_async_engine,
)

engine = create_async_engine(settings.database_url, pool_pre_ping=True)
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)
```

Sync equivalent: `create_engine(...)` + `sessionmaker(...)` with a
`postgresql+psycopg://` URL.

## Session dependency (FastAPI)

```python
from collections.abc import AsyncIterator

from fastapi import Depends


async def get_session() -> AsyncIterator[AsyncSession]:
    async with SessionLocal() as session:
        yield session


@router.get("/users/{user_id}")
async def get_user(user_id: int, session: AsyncSession = Depends(get_session)):
    return await session.get(User, user_id)
```

## Conventions

- **Never** rely on `Base.metadata.create_all` for production schema — that's
  Alembic's job (see `references/alembic.md`). `create_all` is acceptable only in
  throwaway tests that don't exercise migrations.
- Type every model and repository/service signature.
- Log data-access events with structlog (bind the entity/operation); don't log
  full rows or secrets.

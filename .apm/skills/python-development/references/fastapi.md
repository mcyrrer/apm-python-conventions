# FastAPI conventions

Default patterns for HTTP APIs. Async-first, typed, Pydantic in and out.

## Settings (pydantic-settings)

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    app_env: str = "local"
    log_level: str = "INFO"
    database_url: str | None = None


settings = Settings()
```

## App factory

Build the app in a factory so tests and workers can construct it cleanly. Call
`configure_logging()` (see `python-development` skill → `references/structlog.md`)
before creating the app.

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI

from .logging import configure_logging
from .routers import health, items


@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: open pools/clients here; shut them down after `yield`.
    yield


def create_app() -> FastAPI:
    configure_logging(settings.log_level)
    app = FastAPI(title="my-service", lifespan=lifespan)
    app.add_middleware(RequestLoggingMiddleware)
    app.include_router(health.router)
    app.include_router(items.router, prefix="/items", tags=["items"])
    return app


app = create_app()
```

## Routers, models, endpoints

- One `APIRouter` per resource under `routers/`.
- Pydantic models for request and response bodies; annotate `response_model`.
- Endpoints are `async def`. Inject shared resources with `Depends` (settings, DB
  session, clients) — never reach for module globals.

```python
from fastapi import APIRouter, Depends
from pydantic import BaseModel

router = APIRouter()


class ItemIn(BaseModel):
    name: str


class ItemOut(BaseModel):
    id: int
    name: str


@router.post("", response_model=ItemOut, status_code=201)
async def create_item(payload: ItemIn, svc: ItemService = Depends(get_item_service)):
    return await svc.create(payload)
```

## Request-logging middleware (structlog)

Bind a request id (and other request context) so every log in the request shares
it, and emit one structured access log per request.

```python
import time
import uuid

import structlog
from starlette.middleware.base import BaseHTTPMiddleware

log = structlog.get_logger()


class RequestLoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        request_id = request.headers.get("x-request-id", str(uuid.uuid4()))
        structlog.contextvars.bind_contextvars(
            request_id=request_id, method=request.method, path=request.url.path
        )
        start = time.perf_counter()
        try:
            response = await call_next(request)
        finally:
            structlog.contextvars.clear_contextvars()
        log.info(
            "http.request",
            status=response.status_code,
            duration_ms=round((time.perf_counter() - start) * 1000, 2),
        )
        response.headers["x-request-id"] = request_id
        return response
```

## Database

For a data layer, provide the SQLAlchemy session via `Depends` — see the
`database-migrations` skill (`references/sqlalchemy.md`).

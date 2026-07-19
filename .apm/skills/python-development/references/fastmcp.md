# FastMCP conventions

Default patterns for building MCP servers with FastMCP.

## Server skeleton

```python
import structlog
from fastmcp import FastMCP

from .logging import configure_logging

configure_logging()          # before anything logs (see structlog reference)
log = structlog.get_logger()

mcp = FastMCP("my-server")
```

## Tools and resources

Define tools and resources with decorators. Type-hint every parameter and return
value — FastMCP derives the tool schema from the signature, so the annotations are
the contract. Write a clear docstring; it becomes the tool description the model
sees.

```python
@mcp.tool
def add(a: int, b: int) -> int:
    """Add two integers and return the sum."""
    log.info("tool.add", a=a, b=b)
    return a + b


@mcp.resource("config://app-version")
def app_version() -> str:
    """Return the running server version."""
    return "1.0.0"
```

- Use `async def` for tools that do I/O (network, DB, filesystem).
- Log an event per tool call with structured key/values; bind stable context
  (e.g. `tool` name) rather than interpolating into the message.
- Keep tool functions thin — delegate real work to plain, testable functions.

## Running / transport

```python
if __name__ == "__main__":
    mcp.run()                      # stdio transport by default
    # mcp.run(transport="http", host="127.0.0.1", port=8000)  # for HTTP
```

Use **stdio** for local/desktop MCP clients; use the **HTTP** transport when the
server is hosted or shared. Select the transport from configuration, not by
editing the call each time.

## Logging

Reuse the same `configure_logging()` as the rest of the project (JSON in
production, console for localhost) — see the `python-development` skill
(`references/structlog.md`). For stdio servers, logs go to stderr, keeping stdout
clean for the MCP protocol.

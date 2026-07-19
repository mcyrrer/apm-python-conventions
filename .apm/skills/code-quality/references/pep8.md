# PEP 8 cheat-sheet (enforceable subset)

The rules the team enforces, distilled from
[PEP 8](https://peps.python.org/pep-0008/). Ruff mechanically enforces most of
these; this is the "why" and the quick lookup. Ruff's formatter owns layout and
whitespace, its `E`/`W` (pycodestyle) and `I` (isort) rules own the rest.

## Layout

- **Indentation:** 4 spaces per level, never tabs.
- **Line length:** ≤ 79 chars for code, ≤ 72 for docstrings and comments.
- **Blank lines:** two between top-level defs, one between methods; blank lines
  inside a function separate logical steps.
- **Line breaks:** break long lines inside parentheses (implicit continuation),
  and break *before* binary operators:

  ```python
  total = (
      base_price
      + shipping
      - discount
  )
  ```

## Naming

| Kind | Convention | Example |
|------|------------|---------|
| Function, variable, module | `snake_case` | `fetch_user`, `first_name` |
| Class | `CapWords` | `UserAccount` |
| Constant | `ALL_CAPS` | `MAX_RETRIES` |
| Exception | `CapWords` + `Error` | `PaymentDeclinedError` |
| "Internal" | leading underscore | `_cached_value` |

Avoid `l`, `O`, `I` as single-character names (they read as `1`/`0`).

## Imports

- One import per line; put all imports at the top of the file, after the module
  docstring.
- Group in order, blank line between groups: **standard library → third-party →
  local application**.
- No wildcard imports (`from module import *`).

  ```python
  import os
  from datetime import datetime

  import httpx
  from fastapi import FastAPI

  from myapp.config import settings
  ```

## Whitespace

- No space inside `()`, `[]`, `{}`, or immediately before `,`, `;`, `:`.
- One space around binary operators and around `=` in assignments — but **no**
  spaces around `=` for keyword arguments / defaults: `def f(x, y=0):`.
- One space after commas: `func(a, b, c)`.

## Programming recommendations

- Compare to singletons with `is` / `is not`: `if value is None:`.
- Use `isinstance(obj, T)` rather than `type(obj) == T`.
- Don't write bare `except:` — catch specific exception types.
- Use `with` for resource management (files, locks, sessions).
- Prefer `def` over assigning a `lambda` to a name.
- For triple-quoted strings, use double quotes: `"""..."""`.

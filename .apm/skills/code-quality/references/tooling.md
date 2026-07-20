# Ruff + mypy setup

Copy-paste config and commands for the project's linting, formatting, and
type-checking defaults. Ruff replaces Black, Flake8, isort, and pydocstyle in one
fast tool; mypy handles static typing.

## `pyproject.toml`

Add `ruff` and `mypy` to the dev dependency group (`uv add --dev ruff mypy`),
then add these blocks. Adjust `target-version`, `line-length`, and the mypy
`files` path to the project.

```toml
[tool.ruff]
line-length = 79            # PEP 8 code line limit
target-version = "py314"
src = ["src"]

[tool.ruff.lint]
# A pragmatic, PEP 8-aligned rule set:
#   E/W pycodestyle · F pyflakes · I isort · N naming · UP pyupgrade
#   B bugbear · SIM simplify · C4 comprehensions · PL pylint
select = ["E", "W", "F", "I", "N", "UP", "B", "SIM", "C4", "PL"]

[tool.ruff.lint.pydocstyle]
convention = "google"

[tool.ruff.format]
docstring-code-format = true

[tool.mypy]
python_version = "3.14"
files = ["src"]
strict = true
warn_unused_ignores = true
warn_redundant_casts = true
```

## Commands

```bash
uv run ruff format         # apply formatting (Black-equivalent)
uv run ruff format --check # verify formatting without writing (CI)
uv run ruff check --fix    # lint and auto-fix what's safe
uv run ruff check          # lint only, no writes (CI)
uv run mypy                # type-check (reads [tool.mypy])
```

Run `uv run ruff format` then `uv run ruff check --fix` then `uv run mypy` before
committing.

## How the tools map to the rules

| Rule area | Enforced by |
|-----------|-------------|
| Layout, line length, whitespace, quotes | `ruff format` + `E`/`W` |
| Import grouping/order | `ruff check` `I` (isort) |
| Naming conventions | `ruff check` `N` (pep8-naming) |
| Unused imports/vars, undefined names | `ruff check` `F` (pyflakes) |
| Simplifiable code, early returns | `ruff check` `SIM`, `B`, `PL` |
| Modern syntax | `ruff check` `UP` (pyupgrade) |
| Type correctness, missing hints | `mypy --strict` |

Avoid blanket suppressions. When a `# noqa: <code>` or `# type: ignore[<code>]`
is genuinely needed, scope it to the specific rule and add a short reason.

## CI

`assets/lint.yml` runs `ruff format --check`, `ruff check`, and `mypy` on every
push/PR. It installs deps with `uv sync` and targets Python 3.14. Copy it to
`.github/workflows/lint.yml`; adjust the install step only if the project doesn't
use uv.

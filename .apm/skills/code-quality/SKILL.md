---
name: code-quality
description: >-
  Use when setting up linting, formatting, or type-checking for a Python
  project, enforcing PEP 8, or reviewing Python for clean-code compliance.
  Applies this project's defaults — Ruff for formatting and linting (replacing
  Black/Flake8/isort), mypy for type-checking, ≤79-char PEP 8 lines — with
  copy-paste config, and always installs a lint GitHub Actions workflow so style
  and type violations are CI-gated.
---

# Code quality

Set up and enforce clean, PEP 8-compliant Python the way this project
standardizes it. Keep the always-on rules in `clean-code.instructions.md` in
mind; this skill is the deeper "how" — the tooling that makes those rules
mechanical — with runnable config in `references/`.

## Defaults at a glance

- **Formatting + linting:** `ruff` — one tool that formats and lints, replacing
  Black, Flake8, isort, and pydocstyle. Run it in check mode in CI, fix mode
  locally.
- **Type-checking:** `mypy` in a strict-leaning config; type hints on every
  public signature, kept clean.
- **Style baseline:** PEP 8 — 4-space indent, ≤79-char lines, stdlib/third-party/
  local import groups (Ruff's isort rules enforce ordering).
- **CI gate:** a lint workflow runs `ruff format --check`, `ruff check`, and
  `mypy` on every push/PR; setup isn't done until it's in place.

## Setting up code quality

1. **Add the dev dependencies.** Add `ruff` and `mypy` to the project's dev
   dependency group in `pyproject.toml`.
2. **Add the config.** Copy the `[tool.ruff]`, `[tool.ruff.lint]`, and
   `[tool.mypy]` blocks from `references/tooling.md` into `pyproject.toml` and
   adjust `line-length`, target version, and package paths.
3. **Run the tools.** `ruff format` then `ruff check --fix` to auto-fix, then
   `mypy` to type-check. Resolve everything before committing — no blanket
   `# noqa` / `# type: ignore` without a written reason.
4. **Install the CI gate.** Copy `assets/lint.yml` to
   `.github/workflows/lint.yml` in the consumer repo and adjust the install step
   and Python version. This runs `ruff format --check`, `ruff check`, and `mypy`
   so violations can't merge.
5. The clean-code and PEP 8 rules these tools enforce live in
   `clean-code.instructions.md` — read `references/pep8.md` and
   `references/clean-code.md` for the rationale and before/after examples.

## References (load as needed)

- `references/pep8.md` — the enforceable PEP 8 cheat-sheet: layout, the naming
  table, import ordering, whitespace, and programming recommendations, with
  short examples.
- `references/clean-code.md` — clean-code principles with before/after Python
  snippets (single responsibility, argument limits, no flag/output args, early
  returns, explanatory constants, specific exceptions, Pythonic idioms).
- `references/tooling.md` — copy-paste `pyproject.toml` Ruff + mypy config, the
  `ruff format` / `ruff check` / `mypy` commands, and how each maps to the rules.
- `assets/lint.yml` — the ready-to-copy lint/format/type CI workflow.

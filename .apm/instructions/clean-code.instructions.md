---
description: Always-on clean-code & PEP 8 rules — style, naming, functions, error handling, Ruff/mypy enforcement.
applyTo: "**/*.py"
---

# Clean code & PEP 8

Write every line of Python in this project to be clean and PEP 8-compliant by
default — readable, small, and idiomatic. These are the always-on rules; the
`code-quality` skill is the setup recipe that installs the tooling
(Ruff + mypy + a lint CI workflow) which mechanically enforces them, with
copy-paste config in its `references/`.

## Style (PEP 8)

- **4-space indentation**, spaces not tabs. Never mix the two.
- **Keep lines ≤ 79 characters** (≤ 72 for docstrings and comments). Break long
  expressions across lines and break *before* binary operators.
- **Blank lines carry structure.** Two blank lines between top-level
  functions/classes, one between methods; use single blank lines inside a body
  to separate logical steps.
- **Imports at the top**, one module per line, grouped **stdlib → third-party →
  local** with a blank line between groups. Never use wildcard imports
  (`from x import *`).
- **Whitespace:** no space inside `()`/`[]`/`{}` or before `,`/`;`/`:`; surround
  binary operators and `=` (for assignment) with a single space.

## Naming

- **`snake_case`** for functions, variables, and modules; **`CapWords`** for
  classes; **`ALL_CAPS`** for constants; **`Error`-suffixed `CapWords`** for
  exceptions.
- **Intention-revealing names.** A reader should know what a name holds without
  chasing it — `first_name`, not `fna`; `city_count`, not `c`. Prefer nouns for
  values, verbs for functions (`calculate_total`, `fetch_user`).
- **No magic numbers or strings.** Name them as constants
  (`MAX_RETRIES = 3`), don't inline unexplained literals.
- **Avoid the ambiguous single chars** `l`, `O`, and `I` as names, and keep
  vocabulary consistent (don't mix `get_*` with `fetch_*` for the same action).

## Functions

- **One responsibility per function.** If the name needs an "and", or you can
  extract another well-named function from it, split it. Keep functions short
  enough to grasp at a glance.
- **≤ 3 arguments.** Past that, group related parameters into a `dataclass`,
  `TypedDict`, or Pydantic model rather than growing the signature.
- **No boolean/flag parameters.** A flag that switches behaviour is two
  functions hiding in one — write `uppercase(text)` / `lowercase(text)`, not
  `transform(text, upper=True)`.
- **No output arguments or hidden side effects.** Return new values instead of
  mutating what was passed in or reaching out to globals/files unexpectedly.
- **Minimize nesting.** Prefer early returns / guard clauses and helper
  functions over deep `if`/`else` pyramids; aim for one or two levels.

## Idioms & correctness

- **Write Pythonic code:** comprehensions over manual accumulation loops,
  generators for large sequences, context managers (`with`) for resources, and
  built-ins (`sum`, `any`, `enumerate`, `zip`) over hand-rolled equivalents.
- **Compare singletons with `is`/`is not`** (especially `x is None`), and test
  types with `isinstance()` rather than `type(x) == ...`.
- **Catch specific exceptions** (`ValueError`, `KeyError`, custom domain
  errors) — never a bare `except:`. Define custom exceptions for domain errors.
- **DRY and KISS.** Give each piece of knowledge one home; prefer the simplest
  design that works over speculative abstraction.

## Comments & hygiene

- **Docstrings on public modules, classes, and functions** — say what they do,
  their parameters, and what they return. Use double-quoted `"""triple"""`
  strings.
- **Comments explain *why*, not *what*.** Delete noise that restates the code;
  let clear names and structure do the explaining.
- **No dead or commented-out code.** Delete it — version control is the history.

## Enforcement

- **Ruff formats and lints; mypy type-checks.** Keep both clean — no lint
  suppressions or type ignores without a written reason. Add type hints on every
  public signature. Set the tooling up (and the CI gate) with the `code-quality`
  skill.

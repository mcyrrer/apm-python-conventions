# CLAUDE.md — working in this repository

This is an **APM producer repository** (Agent Package Manager,
<https://microsoft.github.io/apm/producer/>). It authors and publishes agent
primitives that consumers install. This repo's package, `python-conventions`,
captures a team's Python backend defaults (structlog, FastAPI, FastMCP,
SQLAlchemy + Alembic on PostgreSQL 17) as always-on instructions plus
load-on-demand setup skills.

## What this repo is

- A **producer** package, not an application. There is no app to run; the
  "product" is the agent context under `.apm/`.
- `apm compile` turns `.apm/` into per-target output (Claude, Copilot, Cursor).
  Producer content must stay valid against the APM spec so compilation is clean.

## Repository map

| Path | What it is | Compiled into agent context? |
|------|------------|------------------------------|
| `apm.yml` | Producer manifest (identity, deps, marketplace). | n/a |
| `.apm/skills/` | Skills (`<name>/SKILL.md` + `references/`, `assets/`). | **Yes** |
| `.apm/instructions/` | Always-/glob-scoped behaviour rules. | **Yes** |
| `docs/` | Human docs & references. | No |
| `README.md` | Marketplace-facing documentation. | No |

**Golden rule:** runtime content for agents goes in `.apm/`; human-only prose
goes in `docs/`. Never mix the two.

## The two skills, at a glance

- **`python-development`** — logging (structlog, JSON/console by env), FastAPI
  HTTP APIs, and FastMCP servers. Deeper "how" with runnable snippets in
  `references/`.
- **`database-migrations`** — SQLAlchemy (typed 2.0 models) + Alembic on
  PostgreSQL 17. Non-negotiable: adding Alembic **always** installs the
  migration-tests CI workflow (`assets/migration-tests.yml`).

The `python-conventions.instructions.md` and `database.instructions.md` files are
the always-on rules; the skills are the on-demand setup recipes. Keep them
consistent — if a default changes, update both the instruction and the skill.

## Before you edit — read this

`docs/apm-format-reference.md` — the granular APM format cheat-sheet: exact
frontmatter for every primitive and the rules that keep changes in-spec.
**Read it before touching anything under `.apm/`.**

## Rules for staying within APM spec

When you add or change anything under `.apm/`, obey the format reference. The
high-frequency rules:

- **Skills:** `SKILL.md` frontmatter needs `name` (kebab-case, ≤64 chars, **must
  equal the parent directory name**) and `description` (≤1024 chars, imperative,
  starts with *when to use it*). Keep `SKILL.md` lean; push detail into
  `references/` (and shippable files into `assets/`) and load it on demand.
- **Instructions:** `*.instructions.md` needs `description` + `applyTo` (a valid
  glob; `**/*` for always-on).
- **Only use documented frontmatter keys.** Inventing fields causes
  compile-time strip warnings.
- **Versioning:** bump `apm.yml` `version` (semver) on every meaningful change
  and mirror it in `marketplace.packages`.

## Adding a new primitive (recipe)

1. Decide the type (skill / instruction) and create it under the matching
   `.apm/` subdirectory using the frontmatter in `docs/apm-format-reference.md`.
2. If it's a skill, name the directory identically to the `SKILL.md` `name`.
3. Cross-link related primitives (skill ↔ instructions) so they compose.
4. Update `README.md` (what's included) and, if publishing, `apm.yml`
   `version` + `marketplace`.
5. Run the verification checklist at the bottom of
   `docs/apm-format-reference.md`.

## Validation

If the `apm` CLI is available, run `apm compile` (and `apm pack` before
publishing) and resolve any stripped-key / placement warnings. If it isn't
available, self-check every changed file against the format reference's checklist
before committing.

## Git / contribution conventions

- Develop on a feature branch; keep commits focused with clear messages.
- Keep instructions and skills in sync — a changed default belongs in both.
- Don't add runtime primitives to `docs/`, and don't add human-only prose to
  `.apm/`.

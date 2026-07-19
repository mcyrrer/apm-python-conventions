# Contributing

This is an [APM](https://microsoft.github.io/apm/producer/) **producer**
repository. Contributions add or refine agent primitives under `.apm/`.

## Before you start

Read, in order:

1. [`CLAUDE.md`](CLAUDE.md) — how the repo is structured and how to work in it.
2. [`docs/apm-format-reference.md`](docs/apm-format-reference.md) — the exact APM
   format rules and frontmatter for every primitive.

## Ground rules

- **Runtime content → `.apm/`. Human docs → `docs/`.** Never mix them.
- **Stay in-spec.** Use only documented frontmatter keys; follow the naming and
  glob rules in the format reference.
- **Skill dir name must equal its `SKILL.md` `name`** (kebab-case, ≤64 chars).
- **Keep instructions and skills in sync** — a changed default belongs in both
  the always-on instruction and the on-demand skill.

## Workflow

1. Branch from the default branch.
2. Make focused changes with clear commit messages.
3. If publishing a change, bump `version` in `apm.yml` (semver) and mirror it in
   `marketplace.packages`.
4. Validate: run `apm compile` (and `apm pack` before release) if the CLI is
   available, and resolve any warnings. Otherwise self-check against the
   verification checklist in `docs/apm-format-reference.md`.
5. Open a pull request describing what changed and why.

## Reporting issues

Open an issue describing the primitive affected, expected vs. actual agent
behaviour, and (if relevant) the compile target.

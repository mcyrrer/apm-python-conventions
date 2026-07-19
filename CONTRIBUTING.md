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
3. Validate: run `apm compile` (and `apm pack` before release) if the CLI is
   available, and resolve any warnings. Otherwise self-check against the
   verification checklist in `docs/apm-format-reference.md`. (CI runs the same
   `apm compile` check plus a version-consistency check on every PR.)
4. Open a pull request describing what changed and why.

## Releasing

Releases are automated — don't hand-edit the version or create tags manually:

1. Run the **Version bump** workflow (Actions tab), choosing `patch` / `minor` /
   `major`. It rewrites `apm.yml` (top-level `version` **and** the mirrored
   `marketplace.packages[].version`) and opens a `release/vX.Y.Z` PR.
2. Review and merge that PR. On merge, the **Release** workflow tags
   `vX.Y.Z` and publishes a GitHub Release.

## Reporting issues

Open an issue describing the primitive affected, expected vs. actual agent
behaviour, and (if relevant) the compile target.

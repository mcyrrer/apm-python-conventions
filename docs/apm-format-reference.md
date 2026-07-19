# APM format reference (granular)

A working reference for authoring **APM producer** content in this repo, so that
changes stay within the [APM specification](https://microsoft.github.io/apm/).
This is a condensed, practical cheat-sheet — the authoritative source is the
[APM docs](https://microsoft.github.io/apm/producer/).

APM = **Agent Package Manager**: "a dependency manager for AI agents — like npm
for agent context." A **producer** repo authors packages (skills, prompts,
instructions, agents, hooks) that consumers install. `apm compile`
deterministically transforms `.apm/` into per-target output (Claude, Copilot,
Cursor, …).

---

## Repository layout

```
agent-files/
├── apm.yml                    # producer manifest (REQUIRED: name, version)
├── README.md                  # marketplace documentation
├── LICENSE
├── .apm/                      # ALL runtime primitives live here
│   ├── skills/<name>/SKILL.md
│   ├── instructions/*.instructions.md
│   ├── prompts/*.prompt.md
│   ├── agents/*.agent.md
│   └── hooks/*.json
└── docs/                      # human docs (NOT compiled into agent context)
```

Rule of thumb: **anything an agent should load at runtime goes in `.apm/`.**
Everything else (analysis, references like this file) is plain repo docs.

---

## `apm.yml` manifest

Only `name` and `version` are strictly required. `version` must be semver
(`\d+\.\d+\.\d+`). Common producer fields:

| Field         | Notes |
|---------------|-------|
| `name`        | **Required.** Package identifier; normalized to kebab-case in marketplace output. |
| `version`     | **Required.** Semver. |
| `description` | One-line summary. |
| `author`, `license` | `license` is an SPDX expression (e.g. `MIT`). |
| `type`        | `instructions` \| `skill` \| `prompts` \| `hybrid`. Use `hybrid` when the package mixes primitive types. |
| `target`      | Compile targets: `claude`, `copilot`, `cursor`, `opencode`, `codex`, `gemini`, `windsurf`, `kiro`, `agent-skills`. |
| `scripts`     | Named shell commands (`apm run <name>`). |
| `dependencies`| `apm:` / `mcp:` / `lsp:` lists. Keep empty (`[]`) if none. |
| `compilation` | `strategy: distributed|single-file`, `resolve_links`, `source_attribution`, `exclude`. |
| `marketplace` | Publishing metadata. `owner.name` is **required** under `marketplace`. Lists `packages:` with `name` + `source`. |

Keep the top-level `name`/`version`/`description` and the `marketplace` entry in
sync. Bump `version` on every published change.

---

## Primitive formats (exact frontmatter)

Each primitive is a markdown file with YAML frontmatter. **Only documented
frontmatter keys survive compilation** — extra keys (e.g. `author`) may be
stripped and produce warnings.

### Skill — `.apm/skills/<name>/SKILL.md`

```markdown
---
name: my-skill            # lowercase a-z 0-9 and hyphens, 1–64 chars,
                          # no leading/trailing/consecutive hyphens,
                          # MUST equal the parent directory name
description: >-           # ≤1024 chars, imperative, starts with a trigger
  Use when <situation>. <what it does>.
---

# Body (keep it lean — target < ~500 lines / 5000 tokens)
Put heavy detail in references/ and tell the agent to load it on demand.
```

Skill directory may also contain `references/`, `scripts/`, `assets/`,
`examples/`. The **directory name is the identity** — if `name` and the directory
disagree, the directory wins.

`description` is load-bearing: it's what the agent matches on to auto-invoke the
skill. Make it start with when to use it ("Use when the user asks to review a
PR…"), not with a description of the skill itself.

### Instructions — `.apm/instructions/<name>.instructions.md`

```markdown
---
description: One-line summary used in compiled context indexes.
applyTo: "**/*.py"      # glob or comma-separated globs; load-bearing.
                        # Commas inside brace alternation {a,b} are NOT list separators.
---
- Long-lived behaviour rules / conventions go here as plain markdown.
```

The deployed filename stem is the basename minus the `.instructions.md` double
extension. Use `applyTo: "**/*"` for always-on rules.

### Prompt — `.apm/prompts/<name>.prompt.md`

Five preserved keys only:

```markdown
---
description: One-line summary; shown in command pickers. (Required for discoverability.)
input:
  - arg_name: "description of the argument"
allowed-tools: [Bash, Read, Grep]   # Claude/Cursor honour this; Copilot ignores it.
model: <slug>                        # optional; Claude/Cursor only.
argument-hint: "<hint>"              # optional; auto-derived from input when omitted.
---
# Body — reference inputs with ${input:arg_name}
```

`author`, `mcp`, and other non-listed keys are stripped on most targets.

### Agent — `.apm/agents/<name>.agent.md`

```markdown
---
name: my-agent                # recommended; defaults to filename stem
description: <required>       # used by harnesses to surface the agent
model: <slug>                # optional; pinned model
tools:                       # optional allow-list
  Read: true
  Grep: true
color: "#3366ff"             # optional
handoffs: [other-agent]      # optional
---
System prompt / persona instructions in the body.
```

### Hook — `.apm/hooks/<event>.json`

Plain JSON event handlers (e.g. `pre-commit`, `on-tool-use`). No frontmatter.

---

## Authoring rules to stay in-spec

1. **Runtime content → `.apm/`; human docs → `docs/`.** Never put analysis prose
   inside a skill body; link to it and keep the skill lean.
2. **Skill `name` == directory name**, kebab-case, ≤64 chars.
3. **Only use documented frontmatter keys.** Don't invent fields.
4. **`description` on skills is a trigger**, imperative and situational.
5. **`applyTo` is required on instructions** and must be a valid glob.
6. **Prompts reference inputs** with `${input:name}` and declare them under
   `input:`.
7. **Bump `apm.yml` `version`** (semver) on every meaningful change, and mirror
   it in `marketplace.packages`.
8. **Validate before publishing:** run `apm compile` / `apm pack` locally and
   fix any stripped-key or placement warnings.

---

## Quick verification checklist

- [ ] `apm.yml` has `name` + valid semver `version`.
- [ ] Every skill dir name matches its `SKILL.md` `name`.
- [ ] Every skill/prompt `description` is present and ≤1024 chars.
- [ ] Every instructions file has `description` + `applyTo`.
- [ ] No undocumented frontmatter keys.
- [ ] `docs/` contains no runtime primitives; `.apm/` contains no human-only prose.

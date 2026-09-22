# rearchitect

A single Agent Skill — packaged for **both Claude Code and OpenAI Codex** — that turns a
**described architectural problem** into **up to three applicable design patterns**, each with an
**interface sketch** of the refactor it would produce, then a recommendation.

You describe code that's painful to change, extend, test, or reason about. `rearchitect` diagnoses
the smell against SOLID and a smell catalog, names only the Gang-of-Four patterns whose triggers
genuinely fire, sketches what each refactor's boundary would look like in your language, and
recommends one — with "leave it as it is" always on the table.

It proposes options to accept or reject. It does **not** implement the refactor, audit your whole
codebase, or force a pattern where none fits.

## What it produces

```
## Problem (as a cost)      — what change is hard, stated concretely
## Diagnosis                — the lenses that fired (SOLID, smells, leaks, depth)
## Candidates (up to 3)     — each: pattern, interface sketch, what it buys / costs
## Recommendation           — the one to reach for, with the plain shape on the table
```

## When it triggers

Phrasings like: "this class is doing too much", "every new type means editing five files", "I keep
copy-pasting this sequence", "how should I restructure this", "what pattern fits here", "this is hard
to test/extend/change".

## Structure

```
rearchitect/
├── .claude-plugin/                 — Claude Code plugin manifest
│   ├── plugin.json
│   └── marketplace.json
├── .codex-plugin/                  — OpenAI Codex plugin manifest
│   └── plugin.json                 (discovers skills in skills/)
├── skills/rearchitect/             — canonical skill (single source of truth)
│   ├── SKILL.md
│   └── references/
│       ├── pattern-matrix.md       — 23 GoF patterns, one trigger each
│       ├── diagnostic-lenses.md    — SOLID, smell→remedy, leakage tests, depth
│       └── interface-sketches.md   — how to sketch a refactor's boundary
├── .agents/skills/rearchitect ->   — Codex discovery path (symlink → skills/rearchitect)
│      ../../skills/rearchitect      so both hosts share one SKILL.md
└── evals/                          — skill-creator eval set (35 cases, 6 assertions each)
```

The `SKILL.md` frontmatter (`name` + `description`) is the format both hosts share, so one file
drives both. The `.agents/skills/rearchitect` symlink means there is no duplicated content to keep
in sync — edit `skills/rearchitect/` and both hosts see it.

## Using it

### Claude Code

Add the repo as a plugin marketplace, or point your plugin config at this directory. The skill is
discovered under `skills/rearchitect/` and triggers on the phrasings above (or invoke it by name).

### OpenAI Codex

Two ways to install — as a **plugin** (recommended) or as a **bare skill**.

- **As a plugin:** the `.codex-plugin/plugin.json` manifest makes this repo a Codex plugin; its
  `skills` field points at `skills/`, so `rearchitect` is bundled and discovered when the plugin is
  installed. (Manifest path is `.codex-plugin/plugin.json` per the openai/codex spec.)
- **As a bare skill, per-repo:** run Codex from inside a project containing this repo — Codex scans
  `$CWD/.agents/skills` and `$REPO_ROOT/.agents/skills`, so the `.agents/skills/rearchitect` symlink
  is picked up automatically.
- **As a bare skill, global:** symlink it onto your personal path once:
  ```sh
  mkdir -p ~/.agents/skills
  ln -s /absolute/path/to/rearchitect/skills/rearchitect ~/.agents/skills/rearchitect
  ```

Then invoke it explicitly with `$rearchitect`, or let Codex select it implicitly when your prompt
matches the description. Disable it locally via `~/.codex/config.toml`:

```toml
[[skills.config]]
path = "/absolute/path/to/rearchitect/skills/rearchitect/SKILL.md"
enabled = false
```

> Note: the `.agents/skills/rearchitect` entry is a git symlink. On Windows, enable symlink support
> (`git config core.symlinks true`) or copy `skills/rearchitect/` to `.agents/skills/rearchitect/`
> instead.

## Provenance

The diagnostic vocabulary and pattern catalog are adapted from the `using-codebase-design` skill in
the `engineering` plugin, reshaped from "shape one boundary well" into "diagnose a problem and
propose refactor options."

## License

MIT

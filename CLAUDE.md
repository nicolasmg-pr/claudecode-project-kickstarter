# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Claude Code plugin (`project-kickstart`, marketplace `niko-plugins`). It ships skills and agents that other repos install to get clean scaffolding and token-saving habits. There is no build, no app, no test suite — the "code" is Markdown skill/agent definitions plus two JSON manifests.

## Validate

```bash
claude plugin validate . --strict
```

Run this after any change to `.claude-plugin/*.json`, `skills/*/SKILL.md`, or `agents/*.md` frontmatter.

## Structure

- `.claude-plugin/plugin.json` — plugin manifest (name, description, author, keywords)
- `.claude-plugin/marketplace.json` — marketplace manifest (`niko-plugins`), lists this plugin as a source
- `skills/<name>/SKILL.md` — skill definition: YAML frontmatter (`name`, `description`, optional `argument-hint`) + numbered-step instructions
- `skills/<name>/references/` — supporting templates/docs a skill links to by relative path (e.g. `[foo.md](foo.md)`, not `references/foo.md`, since the link is resolved from inside `references/`)
- `agents/<name>.md` — subagent definition: frontmatter (`name`, `description`, `tools`, `model`) + a short system prompt

`plugin.json` has no `version` field — every commit to `main` is treated as a new version; users update via `/plugin update project-kickstart`.

## Conventions specific to this repo

- All generated skill/agent/template content is **English only** — the project-kickoff skill enforces this for downstream projects, and it applies here too (non-English tokenizes ~26-30% worse).
- SKILL.md files are lean and imperative (numbered steps, not prose explanation). Keep that style when editing them.
- Agents in `agents/` are all pinned to `model: haiku` with a minimal `tools` allowlist — they're cheap delegation targets (tests, lint, commits), not general-purpose. Don't broaden their tool access or move them off Haiku without a reason.
- Skills must not encourage roleplay hierarchies (PM/Scrum Master/senior-junior simulation) or reliance on auto-compaction — both are explicit anti-patterns this plugin's `context-hygiene` and `feature-start` skills push back on. Keep new skills consistent with that stance.
- Reference templates (`skills/project-kickoff/references/*.md`) are templates *for other projects'* CLAUDE.md/docs — don't confuse their content rules (e.g. "under 60 lines") with rules for this repo's own files.
- Vendored skills are kept **verbatim** from upstream, each with its `LICENSE.txt` beside its `SKILL.md`: `frontend-design` (anthropics/skills, Apache-2.0) and the eight motion/design skills from emilkowalski/skills (MIT) — `emil-design-eng`, `apple-design`, `animation-vocabulary`, `find-animation-opportunities`, `improve-animations`, `review-animations`, `pick-ui-library`, `prototype`. Do not edit them to match this repo's house style; re-sync from upstream instead.
- Skills authored here that cite external docs put the source URL and fetch date at the top of the reference file (see `skills/caching-strategy/references/`).

## Sources

Built from vault notes: Claude Code Token Saving Tips, Context Learnings, Clear good practice, Project Review – Lessons Learnt, Solving the Training Cutoff Problem, Pulling in Fresh Docs with Web Search, Main Info About Skills, Plugins.

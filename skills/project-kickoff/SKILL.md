---
name: project-kickoff
description: Scaffold a new project with best practices — git init, a correct src/tests/docs folder layout, a curated English CLAUDE.md, and token-saving setup. Use when starting a new project, repo, or app from scratch, or when asked to kickstart or bootstrap a project.
argument-hint: "[project-name or short description]"
---

# Project Kickoff

Scaffold a new project so it starts with good structure and low token overhead. Everything generated must be in English — non-English guidelines tokenize ~26-30% worse.

## Step 0 — Gather (one round of questions max)

Ask only what you cannot infer: project name, stack/language, package manager, test/lint commands. Do not block on unknowns — leave `TODO` markers.

## Step 1 — Git

- `git init` if there is no repo. Create a stack-appropriate `.gitignore` with `.env` in it from the start.
- If the stack needs config, commit a `.env.example` with placeholder values (`API_KEY=changeme`), never real ones. Compose/CI files reference `${VARS}` — a git-tracked docker-compose with literal secrets is a finding even for local dev.
- Hold the initial commit until scaffolding is done: `chore: project kickstart scaffolding`.

## Step 2 — Folder layout

Create the directory tree before writing any code. Rules and per-stack trees: [references/folder-layout.md](folder-layout.md).

- Python: **src-layout** (`src/<package>/`, `tests/` outside it) unless the project is a single throwaway script. Wire `pyproject.toml` to `src/` and make `pip install -e .` the first setup step.
- Node/TypeScript: `src/` for source, `dist/` gitignored for build output.
- Framework projects: keep the framework's own convention (Next.js `app/`, Django, Rails) — do not force `src/` on top of it.
- Create the empty `src/`, `tests/`, and `docs/` directories now; a layout decided later is a refactor.
- Record the choice in CLAUDE.md under Conventions in one line.

## Step 3 — docs/ folder

Add a README to `docs/` from [references/docs-readme-template.md](docs-readme-template.md). This folder holds small, scoped reference files the agent cannot know from training. Precision, not volume. Never secrets.

## Step 4 — CLAUDE.md

Write it from [references/claude-md-template.md](claude-md-template.md). Rules:

- English only. Lean: target under 60 lines.
- Only this-project content. If a line would be true in every project, cut it — generic guidelines cause context rot: they eat startup context and degrade output.
- Prune the engineering-standards list to the actual stack (drop React lines for a Python CLI, etc.).
- Long or only-sometimes-relevant material goes to `docs/`, not CLAUDE.md.

## Step 5 — Token-saving setup

- Point out the bundled Haiku subagents: `test-runner`, `lint-fixer`, `commit-writer`. Routine work goes to them, not the main model.
- Never simulate a team (PM / Scrum Master / senior-junior roleplay) — it forces useless planning and reporting loops. The user plans; the agent executes.
- Offer, as optional and skippable: RTK + Caveman terminal I/O compression (~78% less token usage combined). Install commands live in this plugin at `skills/context-hygiene/references/rtk-caveman.md`. Requires cargo and Node ≥ 18.

## Step 6 — Working agreements

Confirm with the user and record in CLAUDE.md:

- New feature = `/clear` + new git branch → feature-start skill.
- Framework-specific work = fetch current docs first → fresh-docs skill.
- Long session = guided `/compact` → context-hygiene skill.
- Repeated, expensive, or read-heavy work (LLM calls, DB access, HTTP, builds) = cache or pool it → caching-strategy skill.
- RAG / semantic search in the stack = retrieve wide, rerank narrow, measure → retrieval-quality skill.

## Step 7 — Wrap up

Show the created tree, make the initial commit, and suggest the first action (usually feature-start for the first feature).

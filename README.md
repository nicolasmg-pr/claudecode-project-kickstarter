# project-kickstart

Claude Code plugin that starts every project with clean structure and token-saving practices.

## Contents

Skills:

- **project-kickoff** — scaffold git, a correct `src/`/`tests/`/`docs/` layout, a curated English CLAUDE.md, token-saving setup
- **feature-start** — `/clear` + new branch discipline at every feature boundary
- **fresh-docs** — search → paste → cite current docs before framework work
- **context-hygiene** — token audit: scoped CLAUDE.md, guided `/compact`, delegation, RTK/Caveman
- **caching-strategy** — cache and pool repeated, expensive work: LLM prompt caching, DB connection pooling, HTTP/CDN, memoization
- **retrieval-quality** — RAG precision: retrieve-then-rerank with a cross-encoder, hybrid retrieval, measured before/after
- **frontend-design** — distinctive, intentional visual design guidance for UI work (from anthropics/skills, Apache-2.0)

Motion and design-engineering skills (vendored from [emilkowalski/skills](https://github.com/emilkowalski/skills), MIT):

- **emil-design-eng** — UI polish, component design, animation and transition decisions
- **apple-design** — Apple's fluid-motion and interface principles, translated to the web
- **animation-vocabulary** — name a motion effect from a loose description (crossfade, shared element transition, rubber-banding)
- **find-animation-opportunities** — sweep a UI for what should animate, and what should not
- **improve-animations** — audit a codebase's motion, emit prioritized plans for cheaper agents to execute
- **review-animations** — strict review of motion code against a craft bar (manual invoke only)
- **pick-ui-library** — curated library picks per task instead of hand-rolled components (manual invoke only)
- **prototype** — build several variants of a UI piece behind a live picker (manual invoke only)

Agents (all Haiku, cheap delegation out of the box):

- **test-runner** — runs tests, reports failures only
- **lint-fixer** — fixes mechanical lint violations, never logic
- **commit-writer** — conventional commit message from the staged diff

## Install

From inside Claude Code:

```
/plugin marketplace add nicolasmg-pr/claudecode-project-kickstarter
/plugin install project-kickstart@niko-plugins
```

Working from a local clone instead: `/plugin marketplace add /path/to/claudecode-project-kickstarter`.

Updates: `version` is intentionally unset in `plugin.json`, so every commit to `main` is a new version — update with `/plugin update project-kickstart`.

## Use

Start a new project: say "start a new project" or run `/project-kickstart:project-kickoff my-app`.
The other skills auto-trigger from their descriptions, or invoke them the same way.

## Structure

```
project-kickstart/
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest
│   └── marketplace.json     # marketplace manifest (niko-plugins)
├── skills/
│   ├── project-kickoff/     # + references/ (folder layout, CLAUDE.md & docs/ templates)
│   ├── feature-start/
│   ├── fresh-docs/
│   ├── context-hygiene/     # + references/ (RTK + Caveman install)
│   ├── caching-strategy/    # + references/ (prompt caching, DB pooling)
│   ├── retrieval-quality/   # + references/ (BGE cross-encoder reranker)
│   ├── frontend-design/     # + LICENSE.txt (Apache-2.0)
│   ├── emil-design-eng/     # ┐
│   ├── apple-design/        # │
│   ├── animation-vocabulary/# │ vendored from emilkowalski/skills
│   ├── find-animation-opportunities/  # │ each + LICENSE.txt (MIT)
│   ├── improve-animations/  # │ + AUDIT.md, PLAN-TEMPLATE.md
│   ├── review-animations/   # │ + STANDARDS.md
│   ├── pick-ui-library/     # │
│   └── prototype/           # ┘ + PICKER.md
└── agents/
    ├── test-runner.md
    ├── lint-fixer.md
    └── commit-writer.md
```

Validate after changes: `claude plugin validate . --strict`

## Sources

Built from vault notes: Claude Code Token Saving Tips · Context Learnings · Clear good practice · Project Review – Lessons learnt · Solving the training cut off problem · Pulling in fresh docs with web search · Main info about Skills · Plugins.

## License

MIT

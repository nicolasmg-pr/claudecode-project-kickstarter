# CLAUDE.md template

Fill in, prune sections that do not apply, keep under ~60 lines total. English only.

```markdown
# <Project Name>

<2-3 lines: what this project is and its current goal.>

## Stack

- <language + version, framework + version, DB, hosting>
- Package manager: <npm / pnpm / uv / ...>

## Commands

- Dev: `<command>`
- Test: `<command>`
- Lint/format: `<command>`

## Conventions

- <folder layout, one or two lines>
- <naming conventions that differ from defaults>

## Engineering standards

<!-- Keep only the lines that match this stack -->
- Typed contracts at every boundary (API routes, session state, model settings). TypeScript strict mode on.
- LLM calls return structured outputs against a strict JSON Schema — never hand-parse fenced JSON.
- Check model capability metadata before exposing model parameters (temperature, max tokens) in UI.
- Keep orchestration components small: one concern per hook/controller; extract instead of growing.
- Persistence is explicit: state is account-scoped (server) or clearly labeled browser-local — never silently localStorage.
- React 19+ / Next 16+: React Compiler on; no manual useCallback/useMemo; module-level or lazy-useState clients, never useMemo for client instances.

## Working agreements

- New feature = /clear + new git branch (short kebab-case name).
- Framework work: fetch current official docs first (search → paste → cite into docs/).
- Delegate: test-runner after changes, lint-fixer before commits, commit-writer for messages.
- Long session: guided /compact with explicit keep/discard instructions.

## Boundaries

- Never commit secrets. docs/ is reference material, not keys.
- <anything the agent must not touch>
```

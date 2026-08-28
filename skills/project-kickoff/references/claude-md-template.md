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

- Layout: <src-layout / flat / framework convention> — source in `<dir>`, tests in `<dir>`, build output in `<dir>` (gitignored).
- <naming conventions that differ from defaults>

## Engineering standards

<!-- Keep only the lines that match this stack -->
- Typed contracts at every boundary (API routes, session state, model settings). TypeScript strict mode on.
- LLM calls return structured outputs against a strict JSON Schema — never hand-parse fenced JSON.
- RAG retrieval is retrieve-wide-then-rerank (cross-encoder), never raw top-k vector search; changes are justified with hit rate @5 on a fixed query set.
- Check model capability metadata before exposing model parameters (temperature, max tokens) in UI.
- Keep orchestration components small: one concern per hook/controller; extract instead of growing.
- Persistence is explicit: state is account-scoped (server) or clearly labeled browser-local — never silently localStorage.
- Caching: <layer + key shape + invalidation rule, one line per cache>. Repeated LLM prompts use prompt caching (stable prefix first); DB access goes through a pool, never a per-request connection.
- Deployed services expose `GET /health` returning 200 only after startup completes (migrations run, pool connected); the platform probe points at it — port-open is not readiness.
- Structured JSON logging (pino / structlog) from the first commit, with requestId + tenantId on every entry. No `console.*` / `print` for operational events.
- Endpoints that trigger paid or model work are rate limited per tenant/IP at the HTTP layer; a budget ceiling is a cost control, not DoS protection.
- React 19+ / Next 16+: React Compiler on; no manual useCallback/useMemo; module-level or lazy-useState clients, never useMemo for client instances.

## Working agreements

- New feature = /clear + new git branch (short kebab-case name).
- Framework work: fetch current official docs first (search → paste → cite into docs/).
- Delegate: test-runner after changes, lint-fixer before commits, commit-writer for messages.
- Repeated + expensive + staleness-tolerant work gets cached or pooled; state the TTL or invalidation event when adding one.
- Long session: guided /compact with explicit keep/discard instructions.

## Boundaries

- Never commit secrets — including docker-compose, CI, and IaC files: `${VAR}` references plus a `.env.example` with placeholders. docs/ is reference material, not keys.
- <anything the agent must not touch>
```

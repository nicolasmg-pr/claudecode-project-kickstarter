---
name: caching-strategy
description: Apply caching and connection-reuse techniques wherever a workload is repeated, expensive, or read-heavy. Use when work touches LLM calls, database access, HTTP APIs, builds/CI, or when asked to make something faster, cheaper, or to add a cache.
argument-hint: "[what to cache — endpoint, query, prompt, build step]"
---

# Caching Strategy

Default posture: **when a result is repeated, expensive, and tolerant of some staleness, cache it.** Cache by default at the LLM and database layers — those are the two that dominate cost in this plugin's projects. Everywhere else, prove the repetition first.

## Step 1 — Qualify the target

Cache only if all four hold:

1. **Repeated** — the same inputs recur across requests, turns, or builds.
2. **Expensive** — network round trip, LLM tokens, heavy query, or slow computation.
3. **Deterministic for its key** — same key must mean same value, or staleness must be acceptable.
4. **Staleness-tolerant** — you can name a TTL or an invalidation event.

Fails any of these → do not cache. A wrong cache is worse than a slow call.

## Step 2 — Pick the layer

Work top-down; the cheapest saved call is the one never made.

| Layer | Technique |
|-------|-----------|
| LLM | Prompt caching (stable system prompt / tools / RAG prefix first). See [references/prompt-caching.md](prompt-caching.md) |
| Database | Connection pooling before query caching. See [references/db-pooling.md](db-pooling.md) |
| HTTP client | Conditional requests (`ETag` / `If-None-Match`), keep-alive connections |
| HTTP server | `Cache-Control`, CDN / edge cache, stale-while-revalidate |
| Application | Redis / Memcached for shared cross-process results |
| In-process | Memoization (`functools.lru_cache`, module-level constants) for pure hot functions |
| Build / CI | Dependency and layer caches keyed on the lockfile hash |

## Step 3 — Design the key

- Include **every** input that changes the output. A missing input serves wrong data to the wrong user.
- Prefix with a schema version (`v2:user:123:profile`) so a format change invalidates the old entries instead of misreading them.
- Never key on a mutable object identity or an unstable serialization (unordered dict, unsorted list).

## Step 4 — Choose invalidation up front

Pick one before writing the cache:

- **TTL** — default. Name a number; "forever" is not a TTL.
- **Event / tag** — invalidate on the write that changes the source of truth.
- **Write-through** — update cache and store in the same operation.

## Step 5 — Guard the boundaries

- Never put secrets, tokens, or credentials in a shared cache.
- Per-user data goes in a per-user key or a per-user cache. Never in a global one.
- Cache is not a store: every read must survive a cache miss and a full flush.
- Do not cache error responses or partial results unless the TTL is short and deliberate.

## Step 6 — Measure

Report before/after: latency, cost, hit rate. No measurement → the cache is unjustified. If hit rate is low, fix the key or drop the cache.

## Record it

When a cache is added, add one line to CLAUDE.md under Engineering standards naming the layer, the key shape, and the invalidation rule. Longer rationale goes to `docs/`.

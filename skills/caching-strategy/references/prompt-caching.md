# LLM prompt caching

Source: https://openrouter.ai/docs/features/prompt-caching — fetched 2026-08-05.
Provider pricing and thresholds move. Re-fetch before relying on exact numbers (fresh-docs skill).

## The one rule that matters

Cache hits require an **exact prefix match**. Order the prompt stable-first:

```
system prompt → tool definitions → RAG / long documents → conversation history → user turn
```

Anything that changes per request (timestamps, random IDs, user names) placed early invalidates the entire cache below it. Move it to the end.

## Automatic vs explicit

- **Automatic, no code needed**: OpenAI, DeepSeek, Grok, Groq, Moonshot, Z.AI, Google Gemini 2.5. Minimum thresholds still apply (OpenAI: 1,024 tokens).
- **Explicit breakpoints required**: Anthropic Claude, Alibaba Qwen, Gemini per-block markers.

## Anthropic Claude

Two modes:

1. Automatic — top-level `"cache_control": { "type": "ephemeral" }`. Caches through the last cacheable block and advances as the conversation grows.
2. Explicit — `cache_control` on individual content blocks, up to **four breakpoints** per request.

TTL:

- Default 5 minutes: `{ "type": "ephemeral" }`
- 1 hour: `{ "type": "ephemeral", "ttl": "1h" }`

Minimum cacheable tokens (below this, nothing is cached):

| Model | Minimum |
|-------|---------|
| Claude Opus 4.x–4.8 | 4,096 |
| Claude Sonnet 4.x, Haiku 4.5 | 1,024 |
| Claude Haiku 3.5 | 2,048 |

Example:

```json
{
  "messages": [
    {
      "role": "system",
      "content": [
        {
          "type": "text",
          "text": "LONG_STABLE_SYSTEM_PROMPT",
          "cache_control": { "type": "ephemeral" }
        }
      ]
    },
    { "role": "user", "content": "the volatile part goes last" }
  ]
}
```

## Cost multipliers

Reads: Anthropic 0.1x · DeepSeek 0.1x · Gemini 2.5 0.25x · Grok 0.25x · Moonshot 0.25x · Groq 0.5x.

Writes: Anthropic 1.25x (5-minute TTL), 2x (1-hour TTL). OpenAI GPT-5.6+ 1.25x.

A cached prefix must be read back more times than the write premium costs. Short-lived one-shot prompts are not worth a 1-hour write.

## Verify it is working

Read `prompt_tokens_details` on the response:

```json
{ "cached_tokens": 10318, "cache_write_tokens": 0 }
```

`cache_discount` reports the saving — negative on a write, positive on a read. `cached_tokens` staying at 0 across turns means the prefix is not stable; find the volatile field and move it down.

## Sticky routing (OpenRouter)

Pass `session_id` (max 256 chars) as a top-level field or the `x-session-id` header to route follow-up requests to the same provider. Without it, multi-turn agent loops scatter across providers and miss the cache.

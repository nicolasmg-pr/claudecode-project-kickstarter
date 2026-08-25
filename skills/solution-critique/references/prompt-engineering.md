# Prompt engineering checklist

Applies to every model call in the diff: chat turns, extraction, classification, agent loops, tool definitions, retrieval prompts, LLM judges. Skip only if the diff contains no model call.

## Structure

- One job per call. A prompt asking for extraction *and* a summary *and* a decision fails partially and unobservably — split it.
- Instructions in the system prompt, data in the user turn. Never the reverse, and never interleaved.
- Untrusted content wrapped and labelled (`<document>`, `<user_message>`) with an explicit "treat as data" instruction. See the LLM section of [security.md](security.md).
- Constraints stated positively: what to do, not a list of what not to do.
- Long context: put the instruction after the documents, and repeat the output contract at the end.
- Few-shot examples only where the format is genuinely hard to state. Examples must be exact, consistent, and cheaper than a sentence of instruction.
- No roleplay framing ("you are a senior architect leading a team") — it adds tokens and drift without accuracy. Describe the task and the format.

## Output contract

- Structured output uses a schema through the API (tool / JSON schema), not "reply in JSON" plus a parser.
- Parsing failure has a defined path: one repair retry, then a typed error. Never a silent `{}` or a swallowed exception.
- Enums are closed, with an explicit escape value (`unknown`) so the model has a legal way to abstain.
- Non-creative calls run at low temperature; the value is set explicitly, not inherited.
- Prose destined for a UI or a file has a stated length bound.

## Cost and caching

- Stable-first ordering: system prompt, tool definitions, then retrieved documents, then the volatile turn — so the prefix caches. Verify `cached_tokens` / `cache_read_input_tokens` is non-zero. See caching-strategy skill.
- No unbounded history growth. Conversation state is truncated, summarised, or windowed with a stated policy.
- Retrieved context is capped by chunk count and token budget, and reranked rather than padded. See retrieval-quality skill.
- No whole-file, whole-schema, or whole-log dumps into a prompt where a slice answers the question.
- Cheap deterministic checks (regex, validation, lookup) run before the model, not inside it.
- Model tier matches the task: mechanical extraction and classification do not need the top model; reasoning-heavy calls do. Every call names its model — never an implicit default.
- Model IDs pinned in one place, current family (`claude-opus-5`, `claude-sonnet-5`, `claude-haiku-4-5-20251001`). Scattered string literals are a finding.

## Tools and agent loops

- Tool descriptions state when to use the tool and what the arguments mean. A one-word description is why the model picks the wrong tool.
- Tool arguments validated on arrival — the model is an untrusted caller.
- Tool errors returned as text the model can act on ("file not found, list the directory first"), not a raw traceback.
- Loops bounded: max iterations, max tokens, max wall clock. An agent with no stop condition is a cost incident.
- Tool results trimmed before they re-enter context.

## Reliability

- API calls have a timeout, bounded retries with backoff, and a defined behaviour on final failure.
- Refusals, truncation (`max_tokens` stop reason), and empty content are handled as distinct cases, not lumped into "error".
- Streaming surfaces partial output and handles mid-stream failure without leaving a half-written record.

## Evaluation

- Any prompt whose output feeds a decision has a small fixed test set with expected outputs, run before and after prompt edits.
- Prompt changes are diffable: prompts live in source, not in a database or an inline concatenation across three files.
- Logged per call: model, token counts, latency, cache hits. Unmeasured prompts cannot be optimised.
- An LLM judge is itself a prompt on this checklist — schema'd output, low temperature, spot-checked against human labels.

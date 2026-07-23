---
name: context-hygiene
description: Audit and reduce token usage in the current session and project setup. Use when the session gets long, output quality degrades, costs climb, or when asked to save tokens, compact context, or clean up CLAUDE.md.
---

# Context Hygiene

Audit the session and project setup for token waste. Work the checklist, report findings, apply only the fixes the user approves.

## Checklist

1. **English everywhere.** CLAUDE.md, skills, and prompts in other languages tokenize ~26-30% worse. Offer to translate.
2. **Scoped CLAUDE.md.** Flag generic guidelines, unused skills, and anything not specific to this project — context rot eats startup context and degrades output. Move long, sometimes-relevant material to `docs/`.
3. **Session boundaries.** Topic switch → user runs `/clear`. Long session → guided compaction: draft an explicit `/compact` instruction (what to keep, what to discard) for the user to run. Never rely on auto-compaction.
4. **Delegation.** Tests, lint fixes, commit messages → the bundled Haiku subagents (`test-runner`, `lint-fixer`, `commit-writer`). Subagents start with a clean context on a cheaper model.
5. **No roleplay hierarchies.** Simulating PM / Scrum Master / senior-junior teams forces useless planning and reporting loops. The user plans; the agent executes.
6. **Memory.** A flat `memory.md` is injected in full into every prompt — cost grows with knowledge. Prefer semantic memory via MCP (e.g. Mem0): structured storage, retrieve only what is relevant.
7. **Terminal I/O compression (optional).** RTK compresses tool output by up to 80%; Caveman trims response verbosity; combined ≈ 78% less token usage. Install steps: [references/rtk-caveman.md](references/rtk-caveman.md).

## Output

End with a short report: what was found, what was changed, estimated impact.

---
name: solution-critique
description: Critique a finished implementation across four lenses — usability, security, operability, and prompt engineering — then apply the approved fixes. Use at the end of an implementation phase, before merge or release, or when asked to review, refine, harden, or optimise the final result.
argument-hint: "[what shipped — feature, branch, or path]"
---

# Solution Critique

End-of-phase review. The code works; the question is whether it is **usable, safe, operable in production, and — where it calls a model — well prompted.**

Critique only what this phase actually changed. No architecture rewrites, no unrelated refactors, no praise padding.

## Step 1 — Scope the review

1. Get the diff: `git diff main...HEAD --stat`, or the paths the user named. No branch → review the working tree plus the last commits of this phase.
2. Read the changed files. Read the entry points they touch (route handler, CLI command, component, prompt builder) even if unchanged — a finding usually lives at the boundary.
3. Name the surfaces in one line each: user surface (UI / CLI / API / none), trust boundary (who sends input, who reads output), deploy surface (long-running service / serverless / job / none), model surface (LLM calls, tools, retrieval, or none).
4. State which lenses apply. A lens with no surface is skipped explicitly, not silently: "no LLM call in this diff → prompt-engineering lens skipped."

## Step 2 — Run the three lenses

Work them in order. Each finding needs a `file:line`, a concrete failure, and a fix that fits this codebase.

| Lens | Asks | Checklist |
|------|------|-----------|
| Usability | Can someone use this without reading the source? Wrong input, empty state, slow call, destructive action. | [references/usability.md](usability.md) |
| Security | What does a hostile input do here? Secrets, authz, injection, exposure. | [references/security.md](security.md) |
| Operability | Will this deploy, restart, and be debuggable in production? Readiness, startup order, shutdown, structured logs. | [references/operability.md](operability.md) |
| Prompt engineering | Is every model call cheap, deterministic enough, and hard to hijack? | [references/prompt-engineering.md](prompt-engineering.md) |

Rules while reviewing:

- Reproduce before reporting. Name the input and the observed or traced result. Cannot construct a failing input → drop the finding.
- Speculative severity is not severity. "Could theoretically" → drop it or demote it to Polish.
- Deep security pass on auth, crypto, or payment code → also run the built-in `/security-review`, then merge its findings into this report instead of duplicating them.
- Repeated LLM or DB cost showing up as a finding → caching-strategy skill. Weak retrieval → retrieval-quality skill. UI craft → frontend-design / emil-design-eng.

## Step 3 — Rank

Three buckets, hardest first:

- **Blocker** — data loss, secret exposure, authz bypass, prompt injection with side effects, a flow a normal user cannot complete.
- **Should fix** — wrong or missing error handling, silent failure, unbounded cost or token growth, confusing default, missing confirmation on a destructive action.
- **Polish** — wording, spacing, naming, small ergonomics.

Cap the report at 10 findings. More than 10 → report the 10 that matter and say how many were cut per bucket. Never pad to fill a bucket; an empty bucket is a valid result.

## Step 4 — Report

```
## Solution critique — <scope>

Surfaces: <user> | <trust boundary> | <deploy> | <model>
Lenses: usability, security, operability, prompt-engineering (skipped: <lens> — <reason>)

### Blockers
- `path/to/file.ts:42` — <failure>. Input: <what triggers it>. Fix: <change>.

### Should fix
...

### Polish
...

Cut: <n> polish findings.
```

## Step 5 — Fix

1. Present the report. Do not edit yet.
2. Ask which findings to apply. Default recommendation: all Blockers, then Should-fix.
3. Apply approved fixes one commit per lens, or one per finding if they touch unrelated files.
4. Verify each fix against the input that reproduced it. Then delegate the suite to `test-runner` and mechanical cleanup to `lint-fixer`.
5. Re-report status per finding: fixed / skipped by user / needs a follow-up branch.

## Record it

- Blocker or Should-fix that reveals a standing rule (a validation boundary, an authz helper, a prompt-caching order) → one line in CLAUDE.md under Engineering standards.
- Everything else stays in the commit message. Long rationale goes to `docs/`.
- Findings the user deferred → list them in the PR description, not in CLAUDE.md.

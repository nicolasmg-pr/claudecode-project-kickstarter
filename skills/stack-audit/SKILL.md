---
name: stack-audit
description: Audit an existing Python codebase against a fixed set of architectural constraints (Streamlit frontend, async FastAPI backend, layered structure, uv, python-dotenv, pydantic, file-backed SQLite, frontend/backend separation, Makefile). Read-only — it reports MET / NOT MET with file-and-line evidence, it does not change code. Use when asked to audit, verify, or check a project against required stack constraints.
argument-hint: "[path to repo root — defaults to the current project]"
---

# Stack Audit

Verify a Python codebase against nine fixed architectural constraints. **Do not change any code.** Output is a report, not a diff.

## Step 1 — Inspect

Read, in this order, before judging anything:

1. `pyproject.toml`, `uv.lock`, `requirements*.txt`, `Makefile`, `.env`, `.env.example`, `.gitignore`
2. The source tree: `git ls-files '*.py' | head -100` — note directories, not just names
3. Every FastAPI entry point (`app = FastAPI(`), router module, and `def`/`async def` on decorated routes
4. Every Streamlit module (`import streamlit`)
5. The database layer: connection string, engine creation, `sqlite3.connect(...)` / `create_engine(...)` arguments

Grep first, read second. Do not infer a dependency from an import alone — confirm it in `pyproject.toml`.

## Step 2 — Judge each constraint

| # | Constraint |
|---|------------|
| 1 | Frontend built with Streamlit |
| 2 | Backend built with FastAPI, used **asynchronously** — `async def` route handlers, not only sync ones |
| 3 | Backend has a clear file structure: routers, models, and services separated, not one file |
| 4 | Packaging and dependencies via **uv**: `pyproject.toml` + `uv.lock`, not bare pip / `requirements.txt` |
| 5 | Config loaded from `.env` via **python-dotenv**, not raw `os.environ` reads or hard-coded values |
| 6 | **pydantic** used for data validation and typed models |
| 7 | Data in **SQLite as a single local file** (`.db` on disk) — not in-memory, not hosted |
| 8 | Frontend and backend clearly separated: distinct modules/directories, frontend talks to backend over **HTTP**, never importing backend internals or sharing in-process state |
| 9 | **Makefile** at the repo root standardising at least `install`, `test`, `lint`, `format`, `run` |

For every constraint report:

- The constraint.
- A verdict: **MET** or **NOT MET**.
- The evidence: exact file path plus line number or config key, with the relevant snippet quoted.
- If NOT MET: precisely what is missing or wrong, and what would fix it.

## Step 3 — Evidence rules

- No file path and no quoted snippet → the constraint is **NOT MET**. Absence of evidence is a failing verdict, not an unknown.
- Partial compliance is NOT MET. Say which part holds and which does not.
  - Constraint 2: any `async def` route counts as async use; **all-sync** routes are NOT MET.
  - Constraint 5: `python-dotenv` declared but never called, or `load_dotenv()` present while secrets are still hard-coded, is NOT MET.
  - Constraint 7: `:memory:`, a Postgres/MySQL URL, or a hosted driver is NOT MET. Name the exact connection string.
  - Constraint 8: any frontend `import` reaching into backend packages is NOT MET regardless of an HTTP client existing elsewhere.
  - Constraint 9: a Makefile missing one of the five targets is NOT MET — list the missing ones.
- Do not soften a NOT MET verdict. No "mostly", no "arguably", no credit for intent.
- Do not propose or apply fixes beyond the one-line remedy per failing constraint. No refactor, no edits.

## Step 4 — Report

```
## Stack audit — <repo>

### 1. Streamlit frontend
Verdict: MET
Evidence: `app/ui/main.py:3` — `import streamlit as st`

### 2. FastAPI used asynchronously
Verdict: NOT MET
Evidence: `app/api/routes.py:12` — `def list_items():` (all 6 routes are sync)
Missing: no `async def` handler anywhere. Fix: convert I/O-bound handlers to `async def` and use an async DB driver (`aiosqlite`).

...

### Summary

| # | Constraint | Verdict |
|---|-----------|---------|
| 1 | Streamlit frontend | MET |
| 2 | Async FastAPI | NOT MET |
| ... | | |

### Fix first
1. <highest-priority gap> — <why it blocks the most>
2. ...
```

Rank the "fix first" list by blast radius: constraints that other constraints depend on (4, 8, 7) before local ones (9, 5).

## Scope

Read-only skill. If the user asks for the fixes after the report, that is a separate task — hand off to `project-kickoff` for missing scaffolding (Makefile, layout, uv) or `solution-critique` for quality findings.

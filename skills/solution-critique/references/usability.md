# Usability checklist

Applies to any surface a human drives: UI, CLI, HTTP API, library API. Walk the surface as a first-time user with the wrong input.

## First run

- Can someone run this from the README alone? Missing env var, missing migration, missing seed data → blocker.
- Failure on missing config names the variable and where to set it. `undefined is not a function` is not a message.
- Defaults work without configuration. Every required flag or required prop is a design decision, not a default.

## Input and error paths

- Empty, oversized, wrong-type, and hostile input each produce a specific message, not a stack trace and not a silent success.
- The message says what to do next, not only what went wrong: "expected ISO date, got `12/03`" beats "invalid date".
- Validation happens at the boundary, once, before partial writes.
- Errors surface where the input was given — inline in a form, on the failing flag, in the response body with a stable error code.

## States

Every async surface needs four, not one: **empty, loading, error, success.**

- Empty state says what to do, not "No data".
- Any call over ~300 ms shows progress. Over ~10 s: cancellable, or reports partial progress.
- Failed operations are retryable without re-entering the input.
- Optimistic updates roll back visibly on failure.

## Destructive and irreversible actions

- Delete, overwrite, force-push, migrate, mass-update → confirmation naming the exact target and count ("delete 412 rows in `users`").
- Confirmation defaults to the safe option. CLI: `--force` required, `--dry-run` available.
- Undo, or a stated recovery path. "Cannot be undone" must be shown before, not after.

## Consistency and accessibility

- Naming, ordering, and units match the rest of the codebase. Same concept, same word, everywhere.
- Keyboard reachable, focus visible, focus trapped in modals, focus returned on close.
- Labels tied to inputs; icon-only controls have accessible names.
- Contrast holds in both themes. Never colour alone to carry meaning.
- Respects `prefers-reduced-motion`.
- CLI: readable without colour, pipeable (machine output on stdout, chatter on stderr), non-zero exit on failure.

## Docs

- The README reflects what shipped: flags, endpoints, env vars.
- One copy-pasteable example per new surface.

# Security checklist

Reference: OWASP Top 10 (web) and OWASP Top 10 for LLM Applications — re-fetch before citing specific IDs (fresh-docs skill).

Review posture: assume every input crossing a boundary is hostile and every output may reach a browser, a shell, a database, or a model. Report only what a named input can reach.

## Secrets

- No key, token, password, or connection string in source, tests, fixtures, logs, error messages, or client bundles.
- Secrets read from env or a secret manager at use time, not baked at build time into anything shipped to a client.
- `.env`, credential files, and dumps are gitignored. Committed once → rotate, not just delete.
- Secrets never enter a cache, a prompt, an analytics event, or a crash report.

## Authentication and authorisation

- Every new endpoint, action, and job answers: who may call this? An unauthenticated route is a decision, stated in code.
- Authorisation is checked server-side per request, on the object being touched — not once at login, not in the client, not by hiding the button.
- IDs from the client are never trusted as ownership proof. `userId` comes from the session, not the body.
- Multi-tenant queries are always scoped by tenant in the query itself, not filtered after fetch.
- Sessions: expiry, revoke on logout, rotate on privilege change. Cookies `HttpOnly`, `Secure`, `SameSite`.

## Injection

- SQL: parameterised queries or an ORM binding. No string concatenation, including in `ORDER BY` and `LIMIT` — allowlist those.
- Shell: no user input in a shell string. Argument arrays, no `shell=True`, no `eval`.
- HTML: escaped by default. Every `dangerouslySetInnerHTML` / `v-html` / `|safe` needs a sanitiser and a reason.
- Paths: user input never concatenated into a filesystem path. Resolve, then verify the result stays inside the intended root.
- URLs from users (webhooks, image fetch, redirects): allowlist scheme and host — otherwise it is SSRF and an open redirect.
- Deserialisation: no `pickle`, no unsigned tokens, no user-controlled class names.

## Data exposure

- API responses return named fields, not whole ORM objects — that is how password hashes and internal flags leak.
- Errors to the client are generic; details go to the server log. No stack traces, no SQL, no file paths in production responses.
- Logs carry no PII, tokens, or full request bodies.
- IDOR check: swap the ID to another tenant's object — does it 403 or 200?

## Transport and dependencies

- TLS everywhere; no disabled certificate verification, no `NODE_TLS_REJECT_UNAUTHORIZED=0`.
- CORS lists explicit origins. `*` plus credentials is a finding.
- New dependencies: real project, recent release, no obvious typosquat. Lockfile committed.
- Audit clean of known-exploitable advisories in code paths this diff actually uses.

## Abuse and limits

- Auth, upload, and expensive endpoints are rate limited.
- Request body, upload size, page size, and retry counts are bounded. Unbounded pagination is a denial-of-service and a cost bug.
- Uploads validated by content, not extension; stored outside the web root; never served from the same origin as the app if untrusted.
- Retries bounded with backoff; no infinite retry against a paid API.

## LLM-specific

- Untrusted text (user content, retrieved documents, tool output, web pages) is data, never instruction. Delimited, labelled, and never concatenated into the system prompt.
- A model cannot be the authorisation layer. Every tool it calls enforces permissions itself, with its own allowlist and arguments validated.
- Model output reaching a shell, a query, a filesystem, or `innerHTML` is validated exactly like user input.
- Model output that triggers a side effect (send, delete, pay, post) needs a human confirmation or a hard allowlist.
- Prompts and retrieved chunks carry no other user's data — check the retrieval filter, not just the prompt.

# Operability checklist

Applies only when the diff touches a deployable — a long-running service, serverless function, or scheduled job. Libraries and pure UI skip this lens explicitly.

## Startup and readiness

- A `GET /health` (or `/ready`) route returns 200 only after startup actually completes: migrations run, DB pool connected, required config validated. Port-open is not readiness — platform TCP checks route traffic into instances that still 500 on cold start.
- The platform probe (Cloud Run startup probe, k8s readinessProbe, load balancer health check) is wired to that route, with an initial delay covering migration time.
- Startup order is explicit: validate config → run migrations → connect pool → listen. A service that listens before it can serve is a cold-start 500 generator.

## Shutdown

- SIGTERM handled: stop accepting, finish in-flight requests, close the pool, then exit. Platforms send it on every deploy and scale-down.
- In-flight jobs are idempotent or resumable after a kill.

## Logging

- Operational events go through one structured JSON logger (pino on Node, structlog on Python) — not `console.*` / `print`. Cloud log agents parse JSON stdout into filterable fields; unstructured lines are grep archaeology across tenants.
- Every entry carries the correlation fields the stack has: requestId, tenantId, threadId/jobId. A log line that cannot be joined to a request is noise at scale.
- Severity is real: `error` means someone should look, `warn` means degraded, `info` is state change. Expected conditions are not errors.
- LLM tracing (Langfuse etc.) covers model calls only — DB errors, guardrail violations, credential health failures still need application logs.

## Failure visibility

- Best-effort paths (metrics recording, cache writes, salvage handlers) log their own failures — a swallowed catch block is an invisible outage.
- Retries and timeouts log the attempt count, so a flapping dependency is visible before it hard-fails.

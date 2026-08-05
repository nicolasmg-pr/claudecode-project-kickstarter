# Database connection pooling (SQLAlchemy)

Source: https://docs.sqlalchemy.org/en/20/core/pooling.html — fetched 2026-08-05.

Pooling is connection reuse, not result caching: it removes the TCP + TLS + auth handshake from every request. Do this before adding a query cache — it is cheaper, safe, and never serves stale data.

## Pool classes

| Class | Use for |
|-------|---------|
| `QueuePool` | Default for sync engines. Bounded connection count. |
| `AsyncAdaptedQueuePool` | Automatic under `create_async_engine()`. `QueuePool` is not asyncio-safe. |
| `NullPool` | Serverless / lambda / short-lived processes. No pooling. |
| `StaticPool` | Single connection, e.g. `sqlite:///:memory:` in tests. |
| `SingletonThreadPool` | One connection per thread. Not for production. |
| `AssertionPool` | Debugging — errors on a second concurrent checkout. |

No pool pre-creates connections; they open on first use, so conservative defaults are safe.

## Configuration

```python
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql+psycopg2://user:password@localhost/dbname",
    pool_size=20,               # persistent connections (default 5)
    max_overflow=10,            # temporary connections beyond pool_size (default 10)
    pool_timeout=30.0,          # seconds to wait for a checkout
    pool_recycle=3600,          # recycle connections older than 1 hour
    pool_pre_ping=True,         # test the connection on checkout
    pool_use_lifo=True,         # LIFO: lets idle connections age out
    pool_reset_on_return="rollback",
)
```

Sizing: `pool_size + max_overflow` per process, multiplied by the process count, must stay under the database `max_connections`. Exceeding it turns a fast app into connection-refused errors under load.

## Stale connections

- `pool_pre_ping=True` — pessimistic; one cheap round trip per checkout, handles a database or proxy that dropped the connection.
- `pool_recycle=3600` — required where the server closes idle connections (MySQL `wait_timeout`, most managed proxies). Set it below the server's idle timeout.
- Optimistic alternative: catch `sqlalchemy.exc.DBAPIError` and check `e.connection_invalidated`; the pool refreshes itself.

## fork() / multiprocessing

Pooled connections **must not** cross a `fork()`. Dispose in the child initializer:

```python
def initializer():
    engine.dispose(close=False)   # close=False: parent keeps its connections

with Pool(10, initializer=initializer) as p:
    p.map(work_function, data)
```

Same rule for Gunicorn/uWSGI pre-fork workers and Celery `prefork`. Serverless: use `NullPool` or an external pooler (PgBouncer) instead.

## Observability

```python
engine = create_engine(url, echo_pool="debug")
```

Logs creation, checkout, checkin, and rollback-on-return. Pool events (`connect`, `checkout`, `reset`) are available via `sqlalchemy.event.listens_for` when you need metrics rather than logs.

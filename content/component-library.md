---
title: Component Library
---

Estoria orchestrates the lifecycle of aggregates, but it relies on a set of vendor-specific components to provide the underlying storage features. These components include:
- [Event Stores](#event-stores), for storing events.
- [Snapshot Stores](#snapshot-stores), for storing snapshots of aggregate state.
- [Aggregate Caches](#aggregate-caches), for caching aggregate state in memory or in a distributed cache.

## Officially Supported Implementations

These components are maintained by the Estoria project. They are tested against each Estoria release and subject to a standard suite of acceptance tests.

### Event Stores

| Event Store | Driver | Global Reads | Stream Deletion | Outbox | Details |
|-------------|--------|--------------|-----------------|--------|---------|
| **[KurrentDB](https://github.com/go-estoria/estoria-contrib/tree/main/kurrentdb/eventstore)** | [KurrentDB-Client-Go](https://github.com/kurrent-io/KurrentDB-Client-Go) | Yes | No | — | Streams map closely to Kurrent streams. |
| **[MongoDB](https://github.com/go-estoria/estoria-contrib/tree/main/mongodb/eventstore)** | [mongo-go-driver](https://github.com/mongodb/mongo-go-driver) | Yes | Yes | [Yes](https://github.com/go-estoria/estoria-contrib/tree/main/mongodb/outbox) | A single collection for all events, or events partitioned across collections by a configurable selector. |
| **[PostgreSQL](https://github.com/go-estoria/estoria-contrib/tree/main/postgres/eventstore)** | [pgx](https://github.com/jackc/pgx) | Yes | Yes | [Yes](https://github.com/go-estoria/estoria-contrib/tree/main/postgres/outbox) | Uses a single table for all events. |
| **[SQLite](https://github.com/go-estoria/estoria-contrib/tree/main/sqlite/eventstore)** | [modernc.org/sqlite](https://gitlab.com/cznic/sqlite) | Yes | Yes | — | Uses a single table for all events; no CGO required. |

**Global Reads** and **Stream Deletion** are the event store's [optional capabilities](/docs/components/event-store#optional-capabilities). KurrentDB's "No" for stream deletion is deliberate: its native delete semantics cannot honestly satisfy the interface's contract, so the store does not claim it (the rationale is in [its README](https://github.com/go-estoria/estoria-contrib/tree/main/kurrentdb/eventstore#unsupported-streamdeleter)). **Outbox** links to the store's companion [transactional outbox](/docs/cqrs#the-transactional-outbox) package, where one exists.

### Snapshot Stores

Coming soon.

| Backend | Driver |
|---------|--------|

### Aggregate Caches

| Aggregate Cache | Type | Driver |
|-----------------|------|--------|
| **[bigcache](https://github.com/go-estoria/estoria-contrib/tree/main/bigcache/aggregatecache)** | In-Memory | https://github.com/allegro/bigcache |
| **[freecache](https://github.com/go-estoria/estoria-contrib/tree/main/freecache/aggregatecache)** | In-Memory | https://github.com/coocood/freecache |
| **[Redis](https://github.com/go-estoria/estoria-contrib/tree/main/redis/aggregatecache)** | Distributed | https://github.com/go-redis/redis |
| **[Valkey](https://github.com/go-estoria/estoria-contrib/tree/main/valkey/aggregatecache)** | Distributed | https://github.com/valkey-io/valkey-go |

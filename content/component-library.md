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

| Event Store | Driver | Details |
|-------------|--------|---------|
| **[KurrentDB](https://github.com/go-estoria/estoria-contrib/tree/main/kurrentdb/eventstore)** | https://github.com/kurrent-io/KurrentDB-Client-Go | Streams map closely to Kurrent streams. |
| **[MongoDB](https://github.com/go-estoria/estoria-contrib/tree/main/mongodb/eventstore)** | https://github.com/mongodb/mongo-go-driver | Can choose between a single collection for all events or one collection per aggregate type. |
| **[PostgreSQL](https://github.com/go-estoria/estoria-contrib/tree/main/postgres/eventstore)** | https://github.com/lib/pq | Uses a single table for all events. |

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

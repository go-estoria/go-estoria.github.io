---
title: Component Library
---

## Event Store Implementations

| Backend | Driver | Details |
|---------|----------------|----------------|
| [KurrentDB](https://github.com/go-estoria/estoria-contrib/tree/main/kurrentdb/eventstore) | https://github.com/kurrent-io/KurrentDB-Client-Go | Streams map closely to Kurrent streams. |
| [MongoDB](https://github.com/go-estoria/estoria-contrib/tree/main/mongodb/eventstore) | https://github.com/mongodb/mongo-go-driver | Can choose between a single collection for all events or one collection per aggregate type. |
| [PostgreSQL](https://github.com/go-estoria/estoria-contrib/tree/main/postgres/eventstore) | https://github.com/lib/pq | Uses a single table for all events. |

## Snapshot Store Implementations

Coming soon.

| Backend | Driver |
|---------|----------------|

## Aggregate Cache Implementations

| Backend | Type | Driver |
|---------|------|----------------|
| [bigcache](https://github.com/go-estoria/estoria-contrib/tree/main/bigcache/aggregatecache) | In-Memory | https://github.com/allegro/bigcache |
| [freecache](https://github.com/go-estoria/estoria-contrib/tree/main/freecache/aggregatecache) | In-Memory | https://github.com/coocood/freecache |
| [Redis](https://github.com/go-estoria/estoria-contrib/tree/main/redis/aggregatecache) | Distributed | https://github.com/go-redis/redis |
| [Valkey](https://github.com/go-estoria/estoria-contrib/tree/main/valkey) | Distributed | https://github.com/valkey-io/valkey-go |

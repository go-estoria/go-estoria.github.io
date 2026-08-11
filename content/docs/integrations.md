---
title: Integrations
type: docs
prev: docs/architecture/
next: docs/troubleshooting/
weight: 850
---

Vendor-specific backends live in the [estoria-contrib](https://github.com/go-estoria/estoria-contrib) module. This page covers first-time setup for each; the [Component Library](/component-library) catalogs them, and each backend's README documents its storage format and options in depth.

## Event Stores

### PostgreSQL

Events are stored in a single table, with a companion streams table tracking per-stream offsets. The store takes a [pgx](https://github.com/jackc/pgx) connection pool.

```go
import (
    pgeventstore "github.com/go-estoria/estoria-contrib/postgres/eventstore"
    pgstrategy "github.com/go-estoria/estoria-contrib/postgres/eventstore/strategy"
    "github.com/jackc/pgx/v5/pgxpool"
)

pool, err := pgxpool.New(ctx, "postgres://user:pass@localhost:5432/mydb")
if err != nil { ... }

// The strategy owns the schema; apply it once at deployment time.
strategy, _ := pgstrategy.NewDefaultStrategy()
if _, err := pool.Exec(ctx, strategy.Schema()); err != nil { ... }

eventStore, err := pgeventstore.New(pool)
```

Notable options: `WithStrategy` (custom table names via the strategy's own options), `WithAppendTransactionHooks` (the [outbox](#transactional-outboxes) integration point), `WithTxOptions`, and `WithMaxEventDataBytes`.

### SQLite

The same single-table layout as PostgreSQL, over `database/sql` with the pure-Go [modernc.org/sqlite](https://gitlab.com/cznic/sqlite) driver — no CGO required.

```go
import (
    "database/sql"
    sqliteeventstore "github.com/go-estoria/estoria-contrib/sqlite/eventstore"
    sqlitestrategy "github.com/go-estoria/estoria-contrib/sqlite/eventstore/strategy"
    _ "modernc.org/sqlite"
)

db, err := sql.Open("sqlite", "file:events.db")
if err != nil { ... }

strategy, _ := sqlitestrategy.NewDefaultStrategy()
if _, err := db.ExecContext(ctx, strategy.Schema()); err != nil { ... }

eventStore, err := sqliteeventstore.New(db)
```

### MongoDB

Events are stored in a single collection, or partitioned across collections by a configurable selector. Both strategies keep offset counters in a streams collection (`_streams` by default). **Multi-document transactions require a replica set** — a single-node replica set is sufficient.

```go
import (
    mongoeventstore "github.com/go-estoria/estoria-contrib/mongodb/eventstore"
    mongostrategy "github.com/go-estoria/estoria-contrib/mongodb/eventstore/strategy"
    "go.mongodb.org/mongo-driver/v2/mongo"
    "go.mongodb.org/mongo-driver/v2/mongo/options"
)

client, err := mongo.Connect(options.Client().ApplyURI("mongodb://localhost:27017/?replicaSet=rs0"))
if err != nil { ... }

db := client.Database("myapp")

// Single collection (the default shape):
strategy, _ := mongostrategy.NewSingleCollectionStrategy(client,
    db.Collection("events"),
    db.Collection(mongostrategy.DefaultStreamsCollectionName),
)

// Or partitioned, one collection per stream type (or per stream ID, or custom):
strategy, _ := mongostrategy.NewMultiCollectionStrategy(client, db,
    mongostrategy.CollectionPerStreamType(),
    mongostrategy.WithAutoEnsureIndexes(),
)

eventStore, err := mongoeventstore.New(client, mongoeventstore.WithStrategy(strategy))

// Create the unique event indexes once at deployment time (the analog of Schema()).
// With a selector that creates collections on the fly, use WithAutoEnsureIndexes instead.
if err := eventStore.EnsureIndexes(ctx); err != nil { ... }
```

Collection names beginning with an underscore are reserved for infrastructure (stream counters, outboxes) and are excluded from global reads. If you are upgrading a database written before the counter-based storage format, see the [backfill instructions](https://github.com/go-estoria/estoria-contrib/tree/main/mongodb/eventstore#upgrading-from-derived-offsets) — with the unique indexes in place, an un-backfilled database fails loudly instead of silently rewriting history.

### KurrentDB

Estoria streams map 1:1 to [KurrentDB](https://www.kurrent.io/) streams, and global reads ride the server's `$all` stream.

```go
import (
    kurrenteventstore "github.com/go-estoria/estoria-contrib/kurrentdb/eventstore"
    "github.com/kurrent-io/KurrentDB-Client-Go/kurrentdb"
)

settings, err := kurrentdb.ParseConnectionString("kurrentdb://localhost:2113?tls=false")
if err != nil { ... }

client, err := kurrentdb.NewClient(settings)
if err != nil { ... }

eventStore, err := kurrenteventstore.New(client)
```

`WithStreamPrefix` namespaces one store's streams on a shared cluster (KurrentDB has no databases); `WithReadAllWindowSize` tunes global-read batching. This store deliberately does not implement stream deletion — KurrentDB's native delete semantics cannot honestly satisfy the interface's contract ([rationale](https://github.com/go-estoria/estoria-contrib/tree/main/kurrentdb/eventstore#unsupported-streamdeleter)).

## Transactional Outboxes

The PostgreSQL and MongoDB stores each ship a companion [outbox](../cqrs#the-transactional-outbox) package. Both register as transaction hooks — outbox items commit or roll back atomically with the events — and both run a polling consumer with at-least-once delivery to your handler.

```go
import pgoutbox "github.com/go-estoria/estoria-contrib/postgres/outbox"

ob, _ := pgoutbox.New(pool, func(ctx context.Context, item *pgoutbox.Item) error {
    return publish(ctx, item) // must be idempotent
})

eventStore, _ := pgeventstore.New(pool, pgeventstore.WithAppendTransactionHooks(ob))

go ob.Run(ctx)
```

```go
import mongooutbox "github.com/go-estoria/estoria-contrib/mongodb/outbox"

ob, _ := mongooutbox.New(db.Collection("_outbox"), db.Collection("_outbox_streams"), handler)

eventStore, _ := mongoeventstore.New(client,
    mongoeventstore.WithStrategy(strategy),
    mongoeventstore.WithTransactionHook(ob),
)

_ = ob.EnsureIndexes(ctx)
go ob.Run(ctx)
```

The MongoDB outbox additionally guarantees strict per-stream delivery order via per-stream leases; see [its README](https://github.com/go-estoria/estoria-contrib/tree/main/mongodb/outbox) for the delivery contract.

## Aggregate Caches

Four cache backends implement `aggregatestore.AggregateCache[S]`: [Redis](https://github.com/go-estoria/estoria-contrib/tree/main/redis/aggregatecache) and [Valkey](https://github.com/go-estoria/estoria-contrib/tree/main/valkey/aggregatecache) (distributed), [Freecache](https://github.com/go-estoria/estoria-contrib/tree/main/freecache/aggregatecache) and [BigCache](https://github.com/go-estoria/estoria-contrib/tree/main/bigcache/aggregatecache) (in-process). All follow the same pattern:

```go
import (
    rediscache "github.com/go-estoria/estoria-contrib/redis/aggregatecache"
    "github.com/redis/go-redis/v9"
)

client := redis.NewClient(&redis.Options{Addr: "localhost:6379"})

cache := rediscache.New[Account](client,
    rediscache.WithTTL[Account](5*time.Minute),
)

cachedStore, _ := aggregatestore.NewCachedStore(store, cache)
```

Each cache serializes entity state with a `StateCodec` (JSON by default, overridable with `WithStateCodec`). Choose in-process caches for single-instance applications and distributed caches when multiple instances must share hydration work.

## Observability

The [OpenTelemetry and Datadog wrappers](../telemetry) instrument event stores, aggregate stores, and snapshot stores with tracing and metrics. See the [Telemetry](../telemetry) page for setup.

## Next Steps

- [Getting Started](../getting-started) for the core usage patterns these backends plug into
- [estoria-examples](https://github.com/go-estoria/estoria-examples) for complete runnable applications against each backend
- [Troubleshooting](../troubleshooting) for common setup and runtime issues

---
title: Architecture Overview
type: docs
prev: docs/typeids/
next: docs/integrations/
weight: 840
---

This page is a high-level map of Estoria: how its components fit together, which one owns which responsibility, and where the extension points are.

## Core Concepts

Estoria maps event sourcing concepts onto a small set of Go types:

- **Entity state** — a plain struct of your own (`Account`, `Order`, …) plus a factory function that produces its initial state. Estoria imposes no interface on it.
- **`estoria.DomainEvent[S]`** — a state-changing event. `ApplyTo(state S) S` is total: a persisted event is a fact, and applying one cannot fail.
- **`eventstore.Store`** — persistent storage for event streams (`StreamReader` + `StreamWriter`).
- **`aggregatestore.Store[S]`** — the high-level interface applications use to create, load, and save aggregates.
- **`aggregatestore.New(...)`** — constructs the event-sourced store, the primary implementation, which reconstructs aggregates by replaying their events.

## Component Relationships

```
┌──────────────────────────────────────────────────────────┐
│                    Application Layer                     │
│             (commands, queries, domain logic)            │
└─────────────┬────────────────────────────────────────────┘
              │  New / Load / Save
┌─────────────▼────────────────────────────────────────────┐
│              Aggregate Store  (Store[S])                 │
│                                                          │
│   optional decorators, stackable in any order:           │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐         │
│   │Snapshotting│  │   Cached   │  │  Hookable  │         │
│   └────────────┘  └────────────┘  └────────────┘         │
│                                                          │
│   core implementation:                                   │
│   ┌──────────────────────────────────────────────┐       │
│   │  Event-sourced store (aggregatestore.New)    │       │
│   │  • replays events to load aggregates         │       │
│   │  • appends events to save them               │       │
│   │  • owns domain event <-> bytes (codec)       │       │
│   │  • owns the aggregate type name              │       │
│   └──────────────────────────────────────────────┘       │
└─────────────┬────────────────────────────────────────────┘
              │  ReadStream / AppendStream
┌─────────────▼────────────────────────────────────────────┐
│                Event Store  (eventstore.Store)           │
└─────────────┬────────────────────────────────────────────┘
              │
   ┌──────────┼──────────┬───────────┬───────────┐
   │          │          │           │           │
┌──▼─────┐ ┌──▼───┐ ┌────▼────┐ ┌────▼─────┐ ┌───▼────┐
│Postgres│ │SQLite│ │ MongoDB │ │KurrentDB │ │ Memory │
└────────┘ └──────┘ └─────────┘ └──────────┘ └────────┘
```

## Data Flow

### Read Path: Loading an Aggregate

1. The application calls `store.Load(ctx, id, nil)`.
2. The store composes the aggregate's typed ID from its configured type name and the given UUID, creates the initial state with the factory, and hydrates it.
3. Hydration calls `eventStore.ReadStream(...)` and, for each event, decodes the payload with the store's `DomainEventCodec` and applies it with `ApplyTo`, advancing the aggregate's version.

With decorators in the stack: a snapshotting store first loads the latest snapshot and replays only the events after it; a cached store returns a cache hit without touching the event store at all.

### Write Path: Saving an Aggregate

1. The application queues events on the aggregate with `Append` (nothing is persisted or applied yet).
2. `store.Save(ctx, aggregate, nil)` encodes each queued event with the `DomainEventCodec` and calls `eventStore.AppendStream(...)`, which enforces optimistic concurrency against the aggregate's loaded version.
3. `AppendStream` returns the written events exactly as a subsequent read would — store-assigned IDs, versions, timestamps, and global positions — and the store applies them to the in-memory state.

With decorators: a snapshotting store consults its policy after the save and captures a snapshot when due; a hookable store runs `BeforeSave`/`AfterSave` hooks; a cached store refreshes its entry.

## Ownership Rules

Estoria assigns each serialization and naming fact exactly one owner:

- **Domain events → bytes** is owned by the aggregate store, through its `estoria.DomainEventCodec[S]` (JSON by default, overridable with `WithDomainEventCodec`). Event store backends persist those bytes verbatim, alongside the codec's declared content type — no backend re-encodes a payload it was handed.
- **Entity state → bytes** (for snapshots and caches) is owned by the component doing the storing, through an `estoria.StateCodec[S]` (JSON by default).
- **The aggregate type name** is owned by the aggregate store, supplied once at construction (`aggregatestore.New(eventStore, "account", NewAccount, ...)`). It becomes the type component of every aggregate ID the store composes, which is how streams are addressed in storage.

## Store Decorators

Decorators wrap any `aggregatestore.Store[S]` and can be stacked:

- **[Snapshotting](../components/aggregate-store/snapshotting)** — loads state from snapshots instead of full replay; captures snapshots per a `SnapshotPolicy`.
- **[Cached](../components/aggregate-store/cached)** — serves loads from an `AggregateCache[S]` backend, falling back to the inner store on a miss.
- **[Hookable](../components/aggregate-store/hookable)** — runs lifecycle hooks (`BeforeLoad`, `AfterSave`, …) around the inner store's operations.

```go
store, _ := aggregatestore.New(eventStore, "account", NewAccount,
    aggregatestore.WithEventTypes(AccountOpened{}, FundsDeposited{}))

cached, _ := aggregatestore.NewCachedStore(store, cache)
snapshotting, _ := aggregatestore.NewSnapshottingStore(cached, snapshotStore,
    snapshotstore.EventCountSnapshotPolicy{N: 100})
hookable, _ := aggregatestore.NewHookableStore(snapshotting)
```

## Extension Points

Vendor-specific implementations live in [estoria-contrib](https://github.com/go-estoria/estoria-contrib); the [Component Library](/component-library) catalogs them.

- **Event stores** implement `eventstore.Store`: PostgreSQL, SQLite, MongoDB, KurrentDB. The core library ships an in-memory store for testing. Optional capabilities — [global reads and stream deletion](../components/event-store#optional-capabilities) — are discovered by type assertion.
- **Snapshot stores** implement `snapshotstore.SnapshotStore`. Both current implementations are in core: `snapshotstore/memory` and `snapshotstore/eventstream` (snapshots stored as events).
- **Aggregate caches** implement `aggregatestore.AggregateCache[S]`: Redis, Valkey, Freecache, BigCache.
- **Transactional outboxes** ride the PostgreSQL and MongoDB event stores' transaction hooks for [reliable delivery](../cqrs#the-transactional-outbox) to external consumers.
- **Telemetry wrappers** ([OpenTelemetry, Datadog](../telemetry)) instrument event, aggregate, and snapshot stores without changing their behavior.

Acceptance suites in `eventstore/storetest` and `snapshotstore/storetest` verify implementations against the behaviors Estoria's components rely on; every contrib backend runs them in CI.

## Key Interfaces

| Interface | Purpose | Key methods |
|-----------|---------|-------------|
| `estoria.DomainEvent[S]` | State-changing event | `EventType()`, `New()`, `ApplyTo(S) S` |
| `estoria.DomainEventCodec[S]` | Domain event serialization | `MarshalDomainEvent`, `UnmarshalDomainEvent`, `ContentType` |
| `estoria.StateCodec[S]` | Entity state serialization | `MarshalState`, `UnmarshalState`, `ContentType` |
| `eventstore.Store` | Event persistence | `ReadStream`, `AppendStream` |
| `eventstore.StreamIterator` | Event stream traversal | `Next`, `Close` |
| `aggregatestore.Store[S]` | Aggregate operations | `New`, `Load`, `Hydrate`, `Save` |
| `aggregatestore.AggregateCache[S]` | Aggregate caching | `GetAggregate`, `PutAggregate` |
| `snapshotstore.SnapshotStore` | Snapshot persistence | `ReadSnapshot`, `WriteSnapshot` |

## Further Reading

- [Getting Started](../getting-started) — build a first Estoria application
- [Components](../components) — detailed documentation for each component
- [Integrations](../integrations) — setup guides for the contrib backends
- [CQRS](../cqrs) — read models, projections, and the transactional outbox

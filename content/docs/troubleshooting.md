---
title: Troubleshooting
type: docs
prev: docs/integrations/
weight: 860
---

Common issues encountered when using Estoria, and how to resolve them.

## Stream Not Found

`eventstore.ErrStreamNotFound` means the stream itself does not exist — the contract distinguishes this from a stream that exists but has no events matching the read options, which yields an empty iterator instead.

**A read before any write is the normal first-load case.** A new aggregate has no stream yet; treat not-found as "create":

```go
account, err := store.Load(ctx, accountID, nil)
if err != nil {
    var loadErr aggregatestore.LoadError
    if errors.As(err, &loadErr) && errors.Is(err, eventstore.ErrStreamNotFound) {
        account = store.New(accountID) // first interaction with this aggregate
    } else {
        return err
    }
}
```

Other causes worth checking:

- **Mismatched aggregate type name.** The stream's type component comes from the name given to `aggregatestore.New(eventStore, "account", ...)`. Two stores with different type names address entirely different streams, even for the same UUID.
- **Wrong backend or database.** Confirm the connection string and database/collection names point where the events were actually written.
- **Malformed stream IDs** when reading the event store directly: aggregate stream IDs render as `account_9f4c...` (type name, underscore, UUID). Parse strings with `typeid.Parse`; construct fresh IDs with `typeid.NewV4("account")` or `typeid.New("account", uuid)`.

## Version Mismatch

```
stream version mismatch: expected version 3, got version 5
```

`eventstore.StreamVersionMismatchError` is optimistic concurrency doing its job: the aggregate was saved by someone else after you loaded it. Retry with a fresh load:

```go
for range maxRetries {
    account, err := store.Load(ctx, accountID, nil)
    if err != nil {
        return err
    }

    account.Append(FundsDeposited{Amount: amount}) // re-derive from fresh state

    err = store.Save(ctx, account, nil)
    if err == nil {
        return nil
    }
    if !errors.Is(err, eventstore.StreamVersionMismatchError{}) {
        return err
    }
    // conflict: loop reloads and re-applies
}
```

Two things to keep in mind:

- **Re-derive, don't replay, your intent.** Business logic must run against the freshly loaded state — the command may no longer be valid after the conflicting save.
- **A save can fail *after* its events were appended** (for example while applying them in memory). That error carries the `aggregatestore.ErrEventsAppended` sentinel; recover by discarding the aggregate and reloading, not by retrying the save.

**If retries never succeed on the MongoDB backend**, and the store was migrated from a pre-counter storage format, the offset counters may be missing: the store then reissues offsets that already exist, and the unique event indexes reject every append as a version mismatch. Run the [backfill](https://github.com/go-estoria/estoria-contrib/tree/main/mongodb/eventstore#upgrading-from-derived-offsets) once.

## Codec Errors

Domain events and snapshot state are serialized by codecs — `estoria.DomainEventCodec[S]` for events, `estoria.StateCodec[S]` for state — JSON by default. Failures surface wrapped in the operation that hit them (`aggregatestore.SaveError`, `HydrateError`, or `eventstore.EventMarshalingError` / `EventUnmarshalingError`).

- **Unexported fields silently don't round-trip** with the JSON codecs. Keep event and state fields exported, and unit-test a marshal/unmarshal round trip per event type.
- **Unregistered event types fail hydration.** Every event type that can appear in a stream must be registered with `aggregatestore.WithEventTypes(...)`; the store uses each event's `New()` to instantiate the right concrete type before decoding.
- **Custom codec pointer semantics:** `UnmarshalState(data []byte, dest *S)` receives `dest` already a pointer. Pass it straight through — `json.Unmarshal(data, dest)`, never `&dest`; the double pointer decodes into a discarded temporary.

```go
// WRONG: &dest is **S; the decoded value is lost
func (c Codec[S]) UnmarshalState(data []byte, dest *S) error {
    return json.Unmarshal(data, &dest)
}

// CORRECT
func (c Codec[S]) UnmarshalState(data []byte, dest *S) error {
    return json.Unmarshal(data, dest)
}
```

## Snapshot Issues

**`snapshotstore.ErrSnapshotNotFound` on a first load is normal** — no snapshot exists until the snapshot policy has triggered once. The snapshotting store handles it internally by falling back to full event replay; don't treat it as an error in application code.

- **Snapshots never being taken?** Check the policy: `snapshotstore.EventCountSnapshotPolicy{N: 100}` captures every 100 events, and the policy is consulted only after a successful save.
- **Snapshots being skipped on load?** A snapshot declaring a content type the store's `StateCodec` does not produce is skipped in favor of event replay — expected after changing codecs; the next save writes a fresh snapshot.
- **The eventstream snapshot store** persists snapshots in a companion stream named by suffixing the aggregate's type: aggregate `account_<uuid>` snapshots to stream `accountsnapshot_<uuid>`. Old snapshots can be pruned automatically with `WithMaxSnapshots`.

## Backend-Specific Issues

### PostgreSQL and SQLite

- **`relation "event" does not exist`** (or the SQLite equivalent): the schema was never applied. The strategy owns it — run `strategy.Schema()` against the database once at deployment time.
- The Postgres store takes a **pgx pool** (`pgxpool.New`), not a `database/sql` handle; SQLite takes `database/sql` with the `modernc.org/sqlite` driver (`sql.Open("sqlite", ...)`).

### MongoDB

- **`Transaction numbers are only allowed on a replica set member`**: Estoria's MongoDB store requires multi-document transactions, which MongoDB only supports on a replica set. A single-node replica set is sufficient (`mongod --replSet rs0` + `rs.initiate()`; testcontainers and most compose setups can do this for you).
- **Missing unique indexes** leave appends without their concurrency backstop. Call `EnsureIndexes` at deployment time, or use the strategy's `WithAutoEnsureIndexes` when a collection selector creates collections on the fly.
- **Underscore-prefixed collections are reserved.** The multi-collection strategy excludes them from global reads, and a collection selector that names one is rejected on writes.

### KurrentDB

- **Connection scheme**: use `kurrentdb://host:2113`, with `?tls=false` only for local development — KurrentDB requires TLS by default.
- **Stream deletion is deliberately unsupported** ([rationale](https://github.com/go-estoria/estoria-contrib/tree/main/kurrentdb/eventstore#unsupported-streamdeleter)); a `StreamDeleter` type assertion on this store correctly reports false.
- **Sharing a cluster** between stores or environments? KurrentDB has no databases; use `WithStreamPrefix` to namespace each store's streams.

### Aggregate Caches

- **Stale reads** are the TTL trade-off; tune with the cache's `WithTTL` option.
- **Codec mismatches** between writer and reader instances (for example after changing `WithStateCodec`) surface as cache misses or unmarshal failures — flush the cache when changing codecs.

### Outboxes

- **Items not being delivered**: confirm the outbox is registered with the event store (`WithAppendTransactionHooks` for Postgres, `WithTransactionHook` for MongoDB) and that a consumer (`Run` or `ProcessNext`) is actually running.
- **A MongoDB outbox stream stops delivering**: after `WithMaxRetries` consecutive failures the item is marked failed and its stream halts for operator inspection — later events in that stream wait until the failed item is resolved. Handlers must be idempotent; delivery is at-least-once.

## General Debugging

- **Enable logging:** `estoria.SetLogger(...)` routes Estoria's internal logging to your logger.
- **Instrument the stores:** the [OpenTelemetry and Datadog wrappers](../telemetry) trace every read, append, and iterator advance.
- **Classify errors, don't string-match:** everything Estoria returns supports `errors.Is`/`errors.As` — sentinels like `eventstore.ErrStreamNotFound` and typed wrappers like `aggregatestore.SaveError` compose.

## Further Reading

- [Architecture Overview](../architecture) — who owns which responsibility
- [Integrations](../integrations) — backend setup guides
- [Examples](../examples) — complete runnable applications to compare against

---
title: Event Stream
type: docs
prev: docs/components/
weight: 280
---

The **event stream snapshot store** persists snapshots as events in the same event store that holds your aggregates' event streams — no separate snapshot storage is needed. Each aggregate's snapshots live in a parallel stream whose type name is the aggregate type with a `snapshot` suffix (a `user` aggregate's snapshots live in its `usersnapshot` stream).

## Usage

```go
import "github.com/go-estoria/estoria/snapshotstore/eventstream"

snapshotStore, _ := eventstream.New(eventStore)
```

## Storage format

A snapshot event's body is the snapshot's state payload as produced by the state codec, stored however the backing event store stores event bodies (see [Over the PostgreSQL event store](#over-the-postgresql-event-store)). The aggregate version the snapshot captures rides in event metadata under `estoria.snapshot_version`, and a writer-supplied timestamp, when present, under `estoria.snapshot_timestamp`. Event metadata keys prefixed `estoria.` are reserved for Estoria itself; backends and callers must not write them.

Because this store depends on metadata surviving the round trip through the backing event store, it requires an event store that faithfully persists event metadata — all estoria-contrib event stores do.

## Retention

By default, this store retains every snapshot ever written. A bounded read (loading an aggregate at a specific version) walks the snapshot stream backwards to find the newest snapshot at or below the requested version, which makes time-travel loads efficient — but be aware that snapshot streams grow without bound.

To bound them, use `WithMaxSnapshots`:

```go
snapshotStore, _ := eventstream.New(eventStore, eventstream.WithMaxSnapshots(3))
```

After each snapshot write, snapshots older than the newest `n` are pruned by truncating the snapshot stream. This requires a backing event store that implements `eventstore.StreamDeleter`, checked at construction. Pruning is best-effort: a prune failure is logged, never surfaced, and the next successful write prunes what the failed one missed.

## Over the PostgreSQL event store

The PostgreSQL event store keeps event bodies in a `jsonb` column, and a snapshot written through it is an ordinary append. That limits this store there:

- **JSON payloads only, and not byte for byte.** `jsonb` keeps parsed JSON rather than the bytes the codec wrote: whitespace is normalized, keys are reordered, and duplicate keys collapse. A payload that is not valid JSON, including any binary codec's output, cannot be written at all — and because the snapshotting aggregate store only logs a failed snapshot write, such an aggregate silently never snapshots.
- **Snapshot writes contend with appends.** Each snapshot takes the global position allocator that every append to the event store waits on.
- **Snapshots enter the global event log.** Each snapshot runs the event store's transaction hooks, so a [transactional outbox](/docs/cqrs/#the-transactional-outbox) receives it, and `ReadAll` yields it to projections reading the global log as an event of type `<aggregate type>snapshot`.

The [PostgreSQL snapshot store](/docs/integrations/#snapshot-stores) keeps snapshots in a table of their own and has none of these limits. To move to it, apply its schema and pass it to the snapshotting aggregate store in place of this one. Existing snapshot streams are then ignored: each aggregate's next load replays its full event stream, and its next snapshot lands in the new table. The old snapshot streams stay in the event store until you delete them. If an aggregate's event stream was truncated, its snapshot stream holds state the event stream no longer has: copy its newest snapshot into the new table before switching, or the aggregate will no longer load.

## Upgrading from before v0.7.0

Snapshot events written by versions before v0.7.0 marshaled the whole snapshot envelope into the event body and carry no version metadata. Such events are skipped, never decoded as state. After an upgrade, the first load of each aggregate replays its full event stream, and the next snapshot write self-heals.

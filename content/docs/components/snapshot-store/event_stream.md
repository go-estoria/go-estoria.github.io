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

A snapshot event's body is the snapshot's state payload, exactly as produced by the state codec. The aggregate version the snapshot captures rides in event metadata under `estoria.snapshot_version`, and a writer-supplied timestamp, when present, under `estoria.snapshot_timestamp`. Event metadata keys prefixed `estoria.` are reserved for Estoria itself; backends and callers must not write them.

Because this store depends on metadata surviving the round trip through the backing event store, it requires an event store that faithfully persists event metadata — all estoria-contrib event stores do.

## Retention

By default, this store retains every snapshot ever written. A bounded read (loading an aggregate at a specific version) walks the snapshot stream backwards to find the newest snapshot at or below the requested version, which makes time-travel loads efficient — but be aware that snapshot streams grow without bound.

To bound them, use `WithMaxSnapshots`:

```go
snapshotStore, _ := eventstream.New(eventStore, eventstream.WithMaxSnapshots(3))
```

After each snapshot write, snapshots older than the newest `n` are pruned by truncating the snapshot stream. This requires a backing event store that implements `eventstore.StreamDeleter`, checked at construction. Pruning is best-effort: a prune failure is logged, never surfaced, and the next successful write prunes what the failed one missed.

## Upgrading from before v0.7.0

Snapshot events written by versions before v0.7.0 marshaled the whole snapshot envelope into the event body and carry no version metadata. Such events are skipped, never decoded as state. After an upgrade, the first load of each aggregate replays its full event stream, and the next snapshot write self-heals.

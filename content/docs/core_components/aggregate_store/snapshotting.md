---
title: Snapshotting Aggregate Store
type: docs
prev: docs/core_components/outbox/
weight: 220
---

The **snapshotting store** wraps an existing aggrergate store and provides snapshotting capabilities.

Snapshotting is a method by which periodic shapshots of an aggregate's state are captured and saved so that subsequent loading of the aggregate can start off preloaded state, subsequently loading only the events that were appended after the snapshotted version.

Snapshotting is typically used for decreasing load-time latency for aggregates.

>TODO: add flow diagram

## Usage

A snapshotting aggregate store requires an inner aggregate store (to defer to when hydrating aggregates) and a snapshot store (to load and save snapshots).

```go
// wrap an existing aggregate store with snapshotting capabilitiies
snapshottingStore, _ := aggregatestore.NewSnapshottingStore(store, snapshotter)
```

## Components

The snapshotting aggregate store relies on the following components:

- **An underlying aggregate store.** Because the snapshotting aggregate store can only hydrate aggregates to specific versions (those for which there are snapshots available), it must defer to another aggregate store to completely hydrate an aggregate to its latest version if events have been appended since the most recent snapshot.
- **A snapshot store.** The snapshotting aggregate store needs to know how to save and load snapshots, and for this it uses a [snapshot store](../snapshot_store/).
- **A snapshot marshaler.** By default, snapshots are marshaled as JSON, but this can be changed via an initialization option.

### Snapshot Store

The snapshot store communicates with a storage backend and is responsible for the actual saving and loading of aggregates. The snapshotting aggregate store uses the snapshot store to hydrate aggregates by attempting to load a snapshot for the provided aggregate ID. It also uses it when saving aggregates to save captured snapshots.

### Snapshot Policy

## Options

### Using a Custom Snapshot Marshaler

By default, the snapshotting aggregate store marshals snapshot data as JSON. You can override this behavior by providing an alternative snapshot marshaler using `WithSnapshotMarshaler()`.

```go
import "github.com/go-estoria/estoria/aggregatestore"

type CustomSnapshotMarshaler[E estoria.Entity] struct{}

func (m *CustomSnapshotMarshaler[E]) Marshal(src PT) ([]byte, error)
func (m *CustomSnapshotMarshaler[E]) Unmarshal(data []byte, dest PT) error

snapshottingStore, err := aggregatestore.NewSnapshottingReader(innerStore, snapshotter,
	aggregatestore.WithSnapshotMarshaler(CustomSnapshotMarshaler{}))
```

### Using a Custom Snapshot Reader

By default, the snapshot store provided when creating the snapshotting aggregate store will be used for reading snapshots. You can override this behavior by providing an alternative snapshot reader using `WithSnapshotReader()`.

```go
import "github.com/go-estoria/estoria/aggregatestore"

type CustomSnapshotReader struct{}

func (CustomSnapshotReader) ReadSnapshot(ctx context.Context, aggregateID typeid.UUID, opts snapshotstore.ReadSnapshotOptions) (*snapshotstore.AggregateSnapshot, error) {
	return nil, errors.New("todo: implement custom snapshot reader")
}

snapshottingStore, err := aggregatestore.NewSnapshottingReader(innerStore, snapshotter,
	aggregatestore.WithSnapshotReader(CustomSnapshotReader{}))
```

### Using a Custom Snapshot Writer

By default, the snapshot store provided when creating the snapshotting aggregate store will be used for writing snapshots. You can override this behavior by providing an alternative snapshot writer using `WithSnapshotWriter()`.

```go
import "github.com/go-estoria/estoria/aggregatestore"

type CustomSnapshotWriter struct{}

func (CustomSnapshotWriter) ReadSnapshot(ctx context.Context, aggregateID typeid.UUID) (*aggregatestore.Snapshot, error) {
	return nil, errors.New("todo: implement custom snapshot writer")
}

snapshottingStore, err := aggregatestore.NewSnapshottingWriter(innerStore, snapshotter,
	aggregatestore.WithSnapshotWriter(CustomSnapshotWriter{}))
```
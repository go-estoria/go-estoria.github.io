---
title: Event Store
type: docs
prev: docs/getting-started/
next: docs/components/aggregate-store
sidebar:
  open: true
weight: 110
---

## Events

An **event** is something that has occurred in the past, often with associated information.

An **event stream** is a uniquely-identified, ordered sequence of events.

Estoria represents an event as:

```go
type Event struct {
	ID              typeid.ID
	StreamID        typeid.ID
	StreamVersion   int64
	GlobalPosition  *int64
	Timestamp       time.Time
	Data            []byte
	DataContentType string
	Metadata        map[string]string
}
```

`GlobalPosition` is the event's position in the store's global order, where the backend has one. `DataContentType` is the MIME type of `Data`, declared by the codec that produced the bytes and returned exactly as written. `Metadata` is optional key-value metadata persisted with the event; keys prefixed `estoria.` are reserved for Estoria itself.

## Event Stores

An **event store** reads events from and appends events to event streams.

### Reading Streams

The `ReadStream` method reads events from an event stream, returning a `StreamIterator` that can be used to iterate over the events in the stream.

```go
iter, _ := eventStore.ReadStream(ctx, streamID, eventstore.ReadStreamOptions{}) (StreamIterator, error)

for {
	event, err := iter.Next(ctx)
	if errors.Is(err, eventstore.ErrEndOfEventStream) {
		break
	}

	// process event
}
```

### Appending to Streams

The `AppendStream` method appends one or more events to an event stream.

```go
written, _ := eventStore.AppendStream(ctx, streamID, []*eventstore.WritableEvent{
    {Type: "balancechanged", Data: []byte(`{"amount": 100}`)},
}, eventstore.AppendStreamOptions{})
```

On success, `AppendStream` returns the written events, populated exactly as a subsequent read of the stream would return them: the store-assigned IDs, stream versions, timestamps, and global positions.

### Event Store Implementations

For production applications, Estoria provides a number of vendor-specific event store implementations via the [Component Library](../../../component-library#event-stores).

The core library also includes a simple in-memory event store for testing and prototyping.

Alternatively, you can implement your own event store. Anything implementing `ReadStream` and `AppendStream` can be used as an event store with Estoria:

```go
import (
  "github.com/go-estoria/estoria/eventstore"
  "github.com/go-estoria/estoria/typeid"
)

interface {
    AppendStream(context.Context, typeid.ID, []*eventstore.WritableEvent, eventstore.AppendStreamOptions) ([]*eventstore.Event, error)
    ReadStream(context.Context, typeid.ID, eventstore.ReadStreamOptions) (eventstore.StreamIterator, error)
}
```

The `eventstore/storetest` package provides an acceptance suite that verifies an implementation against the behaviors Estoria's components rely on.

### Optional Capabilities

Beyond reading and appending, an event store may implement optional interfaces. Callers discover support with a type assertion:

- `eventstore.GlobalReader` reads events from all streams in the store's global order via `ReadAll`, resuming from an exclusive position — the substrate for [projections](../../projections) and read models.
- `eventstore.StreamDeleter` deletes a stream outright via `DeleteStream`, or truncates it through a version when `ToVersion` is set.

Each has a dedicated acceptance suite in `eventstore/storetest`.

Every officially supported implementation provides global reads, and all but KurrentDB support stream deletion — KurrentDB's native delete semantics cannot honestly satisfy the interface's contract, so that store deliberately does not claim it. The [Component Library](/component-library#event-stores) lists per-backend support.

Reading and writing event streams are low-level operations in Estoria. Next, we'll see how to create an Aggregate Store that uses an event store to persist aggregates.

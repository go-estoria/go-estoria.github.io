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
	ID            typeid.ID
	StreamID      typeid.ID
	StreamVersion int64
	Timestamp     time.Time
	Data          []byte
}
```

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
_ = eventStore.AppendStream(ctx, streamID, events []*eventstore.WritableEvent{
    {Type: "balancechanged", Data: []byte(`{"amount": 100}`)},
}, eventstore.AppendStreamOptions{}) error
```

### Event Store Implementations

For production applications, Estoria provides a number of vendor-specific event store implementations via the [Component Library](../../../component-library#event-store-implementations).

The core library also includes a simple in-memory event store for testing and prototyping.

Alternatively, you can implement your own event store. Anything implementing `ReadStream` and `AppendStream` can be used as an event store with Estoria:

```go
import (
  "github.com/go-estoria/estoria/eventstore"
  "github.com/go-estoria/estoria/typeid"
)

interface {
    AppendStream(context.Context, typeid.ID, []*eventstore.WritableEvent, eventstore.AppendStreamOptions) error
    ReadStream(context.Context, typeid.ID, eventstore.ReadStreamOptions) (eventstore.StreamIterator, error)
}
```

Reading and writing event streams are low-level operations in Estoria. Next, we'll see how to create an Aggregate Store that uses an event store to persist aggregates.

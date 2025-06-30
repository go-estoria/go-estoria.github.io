---
title: In-Memory Event Store
type: docs
prev: docs/components/aggregate_store/
weight: 120
---

An in-memory event store stores events in memory.

While not suitable for production use, it serves as a reference implementation and can be particularly useful when prototyping new applications or testing custom aggregate store implementations.

The in-memory event store supports a transactional outbox and custom event marshaler.

## Usage

```go
import "github.com/go-estoria/estoria/eventstore/memory"

eventStore, _ := memory.NewEventStore()
```

### Outbox

To enable outbox functionality, provide an outbox when creating the event store:

```go
import (
    "github.com/go-estoria/estoria/eventstore/memory"
)

obox := memory.NewOutbox()
obox.RegisterHandlers(&SomeEvent{}, func(ctx context.Context, item memory.OutboxItem) {
    fmt.Println("handling outbox item:", item)
})

eventStore, _ := memory.NewEventStore(
    memory.WithOutbox(obox),
)
```

### Customer Event Marshaler

To use a custom event marshaler, provide one when creating the event store:

```go
import (
    "github.com/go-estoria/estoria/eventstore/memory"
)

type MyCustomMarshaler[E eventstore.Event] struct {}

func (m MyCustomMarshaler[E]) Marshal()

eventStore, _ := memory.NewEventStore(
    memory.WithEventMarshaler(MyCustomMarshaler{}),
)
```

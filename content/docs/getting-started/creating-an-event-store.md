---
title: Creating An Event Store
type: docs
prev: getting-started/defining-events
next: getting-started/creating-an-aggregate-store
weight: 123
---

The **event store** persists entity events. In a typical application, a database-driven implementation would be used, but for demonstration purposes we will use Estoria's in-memory implementation, which simply keeps events in a slice in memory.

Let's create a new event store for our application:

```go
import "github.com/go-estoria/estoria/eventstore/memory"

eventStore, _ := memory.NewEventStore()
```

Using our event store, we can save and load events. The event store is agnostic to our entity types, so we can use it to store events for any entity type we define.

```go
_ = eventStore.AppendEvents(context.Background(), typeid.NewV4("user"),
    []eventstore.WriteableEvent{
        &UserNameChanged{NewName: "Juliette"},
    },
)
```

Event store implementations for a number of database backends are available in [estoria-contrib](https://github.com/go-estoria/estoria-contrib). You can also implement your own custom event store.

Now that we have an event store that can load and save events, let's create an aggregate store that uses it to load and save aggregates.

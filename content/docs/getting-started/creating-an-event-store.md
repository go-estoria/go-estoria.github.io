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
import (
    "log"
    "github.com/go-estoria/estoria/eventstore/memory"
)

eventStore, err := memory.New()
if err != nil {
    log.Fatalf("failed to create event store: %v", err)
}
```

Using our event store, we can save and load events. The event store is agnostic to our entity types, so we can use it to store events for any entity type we define.

```go
eventStore.AppendEvents(context.Background(), typeid.New("user", user.ID), []eventstore.WriteableEvent{
    &UserCreated{Name: "Paul Atreides"},
    &UserNameChanged{NewName: "Maud'Dib"},
})
```

Event store implementations for a number of database backends are available in [estoria-contrib](https://github.com/go-estoria/estoria-contrib). You can also [implement your own custom event store](advanced/implementing-a-custom-event-store).

Now that we have an event store that can load and save events, let's create an aggregate store that uses it to load and save aggregates.

---
title: Creating An Event store
type: docs
prev: getting-started/defining-events
next: getting-started/creating-an-aggregate-store
weight: 123
---

An event store persists entity events. In a typical application, a database-driven implementation would be used, but for demonstration purposes we will use Estoria's in-memory event store implementation, which simply keeps events in a slice in memory.

Let's create a new event store for our application:

```go
eventStore, err := memoryeventstore.New()
if err != nil {
    log.Fatalf("failed to create event store: %v", err)
}
```

Event store implementations for a number of database backends are available in estoria-contrib. You can also implement your own custom event store.

Now that we have an event store that can load and save events, let's create an aggregate store that will use it to load and save aggregates.

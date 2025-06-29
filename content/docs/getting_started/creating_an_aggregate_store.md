---
title: Creating An Aggregate store
type: docs
prev: getting_started/creating_an_event_store
next: getting_started/working_with_aggregates
weight: 124
---

An aggregate store persists aggregates by using an underlying event store to load and save events. Each aggregate store works with a single aggregate type.

Let's create a new aggregate store for Users:

```go
aggregateStore, err = aggregatestore.NewEventSourcedStore(eventStore, NewUser,
    aggregatestore.WithEventTypes(
        UserNameChangedEvent{},
    ),
)
if err != nil {
    log.Fatalf("failed to create aggregate store: %v", err)
}
```

Notice that we're passing in the event store and the factory function for our User type. This enables the aggregate store to instantiate a User before applying events to it in order to reconstruct its state. We're also telling the aggregate store what types of events can be used with our entity type.

Now that we have an aggregate store for Users, we can begin working with User aggregates.

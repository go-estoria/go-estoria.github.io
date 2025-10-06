---
title: Creating An Aggregate store
type: docs
prev: getting-started/creating-an-event-store
next: getting-started/working-with-aggregates
weight: 124
---

The **aggregate store** persists aggregates by using an underlying event store to load and save events. An aggregate store works with a single aggregate type.

Let's create a new aggregate store for Users in our application:

```go
import "github.com/go-estoria/estoria/aggregatestore"

aggregateStore, _ := aggregatestore.New(eventStore, NewUser,
    aggregatestore.WithEventTypes(
        UserNameChangedEvent{},
    ),
)
```

Notice that we're passing in the event store and the factory function for our User type. This enables the aggregate store to instantiate a User before applying events to it in order to reconstruct its state. We're also telling the aggregate store what types of events can be used with our entity type.

Now that we have an aggregate store for Users, we can begin working with User aggregates.

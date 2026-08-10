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

aggregateStore, _ := aggregatestore.New(eventStore, "user", NewUser,
    aggregatestore.WithEventTypes(
        UserNameChanged{},
    ),
)
```

We're passing in four things: the event store, an **aggregate type name**, the factory function for our User type, and the event types that apply to Users.

The aggregate type name (`"user"`) becomes the type component of every aggregate ID this store composes, which is how event streams are addressed in storage. Choose it once and keep it stable for the lifetime of your data — changing it later would strand existing streams under the old name. It must be non-empty and must not begin or end with an underscore; interior underscores are fine. The same grammar applies to event type names, validated when event types are registered.

The factory function enables the aggregate store to instantiate a User before applying events to it in order to reconstruct its state, and the registered event types tell the store which events it may encounter in a User's stream.

Now that we have an aggregate store for Users, we can begin working with User aggregates.

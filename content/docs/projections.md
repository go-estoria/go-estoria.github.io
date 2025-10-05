---
title: Projections
type: docs
prev: docs/components/aggregate-cache/
next: docs/telemetry/
weight: 800
---

A **projection** is a read model that is built by processing a stream of events. Projections are used to create views of the data that are optimized for querying and reporting.

## Creating Projections

To project an event stream, simply obtain an event stream iterator from your event store and create a new projection. Then, provide an event handler function that will be called for each event in the stream. The event handler function then updates the projection's state based on the event data.

```go
import (
    "context"
    "github.com/go-estoria/estoria/eventstore"
    "github.com/go-estoria/estoria/eventstore/projection"
)

// get a stream iterator
iter, _ := eventStore.ReadStream(ctx, streamID, eventstore.ReadStreamOptions{})

// create a projection
proj, _ := projection.New(iter)

// run the projection
_, err = proj.Project(ctx, projection.EventHandlerFunc(func(ctx context.Context, evt *eventstore.Event) error {
    // handle each event
}))
```

Note that projections operate directly with event streams, not aggregates. Your handler logic must determine event types and deserialize event data as needed.

## In-Memory Projections

Here's a simple example that demonstrates how to create an in-memory projection that calculates the balance of an account by processing `BalanceChanged` events:

```go
// simple projection state
var balance int

// run the projection
_, err = proj.Project(ctx, projection.EventHandlerFunc(func(_ context.Context, evt *eventstore.Event) error {
    if evt.ID.Type != "balancechanged" {
        return nil
    }

    var e BalanceChangedEvent
    event, _ := json.Unmarshal(evt.Data, &e)
    balance += e.Amount
    return nil
}))
```

## Persistent Projections

A projection that updates a persistent read model (e.g., a database table) is called a persistent projection. Persistent projections are typically used in CQRS systems to create read models that can be queried independently of the write model.

Here's the same example as above, but this time updating a SQL database table:

```go
// database connection
var db *sql.DB

// run the projection
_, err = proj.Project(ctx, projection.EventHandlerFunc(func(_ context.Context, evt *eventstore.Event) error {
    if evt.ID.Type != "balancechanged" {
        return nil
    }

    var e BalanceChangedEvent
    event, _ := json.Unmarshal(evt.Data, &e)
    _, err := db.Exec(
        "UPDATE accounts SET balance = balance + ? WHERE account_id = ?",
        e.Amount, 
        evt.ID.EntityID,
    )
    return err
}))
```

## Error Handling

By default, if an error is returned from the event handler function, the projection will stop processing further events and return the error. This behavior can be changed by providing the `WithContinueOnHandlerError(true)` option when creating the projection. In this mode, the projection will log the error and continue processing subsequent events.

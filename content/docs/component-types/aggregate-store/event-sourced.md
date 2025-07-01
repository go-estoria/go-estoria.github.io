---
title: Event-Sourced Aggregate Store
type: docs
weight: 220
---

The event-sourced aggregate store loads and saves aggregates using an event store.

Aggregates are hydrated by streaming events from an event store and applying them to an aggregate's root entity to reconstruct its state. Changes to aggregates are saved by appending events to the event store representing incremental state changes.

This functionality -- the loading and saving of aggregates using event streams -- is central to event-sourcing using Estoria.

## Usage

```go
import (
    "context"
    "github.com/go-estoria/estoria/aggregatestore"
    "github.com/go-estoria/estoria/entity"
    "github.com/google/uuid"
)

func main() {
    // create an event store
    eventStore, _ := eventstore.NewInMemoryStore()

    // create the aggregate store (type parameter inferred from entity factory)
    store, _ := aggregatestore.NewEventSourcedStore(eventStore, NewThing)

    // create an aggregate
    aggregate, _ := store.New(uuid.New())

    // append some events
    aggregate.Append(
        &NameChangedEvent{NewName: "Paul Atreides"},
        &NameChangedEvent{NewName: "Maud Dib"},
    )

    // save the aggregate
    store.Save(context.Background(), aggregate)
}
```

## API Overview

### Creating an Event Sourced Aggregate Store

Creating a new event sourced aggregate store requires two things:
- an **event store**, so that it can save and load events for the aggregate;
- an **entity factory function**, so that it can create new instances of the aggregate's root entity type.

Each aggregate store works specifically with a single aggregate type. Applications working with multiple aggregates should create aggregate stores for each aggregate type.

### Creating a New Aggregate

The `New()` method creates a new aggregate with a default version of 0 (no applied events) and a default root entity using the configured entity factory function.

```go
func New(id uuid.UUID) *Aggregate[E]
```

### Loading an Aggregate

The `Load()` method is a convenience method that instantiates a new aggregate using `New()` and then hydrates it by calling `Hydrate()`.

```go
func Load(ctx context.Context, id uuid.UUID, opts LoadOptions) (*Aggregate[E], error)
```

### Hydrating an Aggregate

Hydrating refers to the process of bringing an aggregate from its current state to some desired latter state.

The `Hydrate()` method streams events from the event store beginning at version N+1, where N is the current version of the aggregate, and applies them to the aggregate root entity.

```go
func Hydrate(ctx context.Context, agg *Aggregate[E], opts HydrateOptions) error
```

### Saving an Aggregate

The `Save()` method appends new events to the event store representing changes to the aggregate root entity.

```go
func Save(ctx context.Context, agg *Aggregate[E], opts SaveOptions) error
```
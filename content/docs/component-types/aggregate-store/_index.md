---
title: Aggregate Store
type: docs
sidebar:
  open: false
weight: 210
---

An **aggregate** is a cluster of domain objects that can be treated as a single unit. The aggregate's root entity is the entrypoint through which we interact with and manage the state and behavior of all related objects.

An **aggregate store** loads, hydrates, and saves aggregates. It is a generic component that requires an entity type as a type parameter:

```go
type AggregateStore[E estoria.Entity] interface {...}
```

This enables it to instantiate new entities and events that are applicable to that entity type. To work with multiple entity types, simply create an aggregate store for each one.

## Aggregate Store Implementations

### Event-Sourced Aggregate Store

The [event-sourced aggregate](./event_sourced) store loads and saves aggregates using an event store.

This functionality – the loading and saving of aggregates via event streams – is central to event sourcing using Estoria.

### Snapshotting

The [snapshotting aggregate store](./snapshotting) wraps an underlying aggregate store and provides the ability to capture snapshots of aggregates at specific versions. These snapshots can then be loaded and applied to the aggregate prior to loading events from the event store.

### Caching

The [cached aggregate store](./cached) wraps an underlying aggregate store and provides caching for aggregates. Caching reduces the need to load aggregates from persistence on every operation.

### Lifecycle Hooks

The [hookable aggregate store](./hookable) wraps an underlying aggregate store and provides the ability to inject lifecycle hooks that are called before and after aggregates are loaded and saved.

### Custom Aggregate Stores

Anything implementing the following interface can be used as an aggregate store with Estoria:

```go
import (
  "github.com/go-estoria/estoria/aggregatestore"
  "github.com/go-estoria/estoria/entity"
  "github.com/go-estoria/estoria/typeid"
)

type AggregateStore[E estoria.Entity] interface {
    New(id uuid.UUID) (*aggregatestore.Aggregate[E], error)
    Load(context.Context, typeid.ID, aggregatestore.LoadOptions) (*Aggregate[E], error)
    Hydrate(context.Context, *aggregatestore.Aggregate[E], aggregatestore.HydrateOptions) error
    Save(context.Context, *aggregatestore.Aggregate[E], aggregatestore.SaveOptions) error
}
```

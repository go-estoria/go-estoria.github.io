---
title: Aggregate Store
type: docs
sidebar:
  open: false
weight: 210
---

An **aggregate** is a group of domain objects that is treated as a single unit. The aggregate's _root_ entity is the entrypoint through which we interact with and manage the state and behavior of all related objects.

An **aggregate store** loads, hydrates, and saves aggregates. It is a generic component parameterized by the entity type it manages:

```go
type Store[S any] interface {...}
```

This enables it to instantiate new entities and events that are applicable to that entity type. To work with multiple entity types, simply create an aggregate store for each one.

## Aggregate Store Implementations

### Event-Sourced Aggregate Store

The [event-sourced aggregate store](./event-sourced) loads and saves aggregates using an event store.

This functionality – the loading and saving of aggregates via event streams – is central to event sourcing using Estoria.

### Snapshotting

The [snapshotting aggregate store](./snapshotting) wraps an underlying aggregate store and provides the ability to capture snapshots of aggregates at specific versions. These snapshots can then be loaded and applied to the aggregate prior to loading events from the event store.

### Caching

The [cached aggregate store](./cached) wraps an underlying aggregate store and provides caching for aggregates. Caching reduces the need to load aggregates from persistence on every operation.

### Lifecycle Hooks

The [hookable aggregate store](./hookable) wraps an underlying aggregate store and provides the ability to inject lifecycle hooks that are called before and after aggregates are loaded and saved.

### Wrapping Aggregate Stores

The `aggregatestore.Store[S]` interface is what the wrapping stores above compose over:

```go
type Store[S any] interface {
    AggregateType() string
    New(id uuid.UUID) *Aggregate[S]
    Load(ctx context.Context, id uuid.UUID, opts *LoadOptions) (*Aggregate[S], error)
    Hydrate(ctx context.Context, aggregate *Aggregate[S], opts *HydrateOptions) error
    Save(ctx context.Context, aggregate *Aggregate[S], opts *SaveOptions) error
}
```

Aggregate construction belongs exclusively to the `aggregatestore` package, so from-scratch third-party implementations of this interface are not supported. To extend aggregate store behavior, wrap an existing store — the snapshotting, cached, and hookable stores above are all examples of this pattern, delegating to an inner store and adding behavior around it. The pluggable surfaces are the event store, the snapshot store, and the aggregate cache.

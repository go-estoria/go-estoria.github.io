---
title: Introduction
type: docs
prev: /docs
next: /docs/getting-started
weight: 110
---

## What is Estoria?

Estoria is an event sourcing toolkit for Go. Event sourcing lets you store database entities as a series of state-changing events. These events are then replayed to reconstruct an entity's current (or past) state.

## Concepts

Estoria is built around a small set of core concepts: entities, events, and aggregates.

### Entities

An **entity** is any uniquely identifiable domain object persisted in a data store. Traditionally, these usually map to rows in tables (relational databases) or documents in collections (document databases).

For example, an `Order` in an order-processing system:

```go
type Order struct {
    ID         uuid.UUID
    CustomerID uuid.UUID
    Status     string
    LastUpdatedAt  time.Time
}
```

### Events

An **event** is something that has happened in the past to an entity.

For example, an `OrderStatusUpdated` event in our order-processing system:

```go
type OrderStatusUpdated struct {
    OrderID     uuid.UUID
    NewStatus   string
    EffectiveAt time.Time
}
```

By storing a temporal sequence of events about an entity, we can reconstruct the entity's current (or previous) state by replaying these events in order.

Events are immutable; we cannot (and should not attempt to) change history. If a correction to the result of a previous event is needed, a new event is added to represent the correction. Preservation of mistakes is a key feature of event sourcing, as it allows for a complete audit trail of changes.

An **event store** is a persistence component that can store and read events.

Estoria provides [officially-supported event store implementations](../component-library/#event-store-implementations) for a variety of storage backends from AWS, Azure, and Google Cloud.

### Aggregates

An **aggregate** is an association of domain objects that is treated as a single unit, and the aggregate _root_ is the domain entity through which all mutations on the aggregate must take place.

For example, an `Order` aggregate might consist of the `Order` entity itself and a collection of `OrderItem` entities that represent the items in the order:

```go
type Order struct {
    ID         uuid.UUID
    CustomerID uuid.UUID
    Status     string
    Items      []OrderItem
    UpdatedAt  time.Time
}

type OrderItem struct {
    ID       uuid.UUID
    SKU      string
    Quantity int
}
```

The `Order` entity is the aggregate root, containing a collection of `OrderItem` entities. Operations such as adding or removing items from the order must be performed through `Order`, theaggregate root, to ensure consistency and integrity of the aggregate.

Estoria provides APIs for saving and loading aggregates using streams of events that represent changes to the aggregate's state. An `Order`, for example, will experience events such as `OrderCreated`, `OrderItemAdded`, and `OrderStatusUpdated` that represent changes made to the order over time.

Estoria doesn't need to know much about your aggregate root entities other than how to identify them and how to serialize and deserialize them. The structure and complexity of your domain objects is yours to decide.

An **aggregate store** is a component that can save and load aggregates.

Estoria's event-sourced aggregate store uses an underlying event store to save and load aggregates.

## Getting Started

See the [Getting Started](../getting-started) guide to learn how to use Estoria in your Go project.

---
title: Getting Started
type: docs
prev: /docs/introduction/
next: docs/components/
weight: 120
---

## Installation

Estoria requires Go 1.23 or above.

```shell
go get github.com/go-estoria/estoria
```

## Usage

See full runnable examples in [estoria-examples](https://github.com/go-estoria/estoria-examples).

## Concepts

Estoria is built around a small set of core concepts: entities, aggregates, and events.

### Entities

An **entity** is any uniquely identifiable domain object that is persisted in a data store. In traditional persistence approaches, these usually map to rows in tables (relational databases) or documents in collections (document databases).

For example, a `User` in an authentication service:

```go
type User struct {
	ID   uuid.UUID
	Name string
}
```

Or an `Order` in an order-processing system:

```go
type Order struct {
	ID         uuid.UUID
	CustomerID uuid.UUID
	Status     string
	UpdatedAt  time.Time
}
```

### Aggregates

An aggregate is a group of domain objects that is treated as a single unit, and the aggregate _root_ is the entity through which all operations on the aggregate must take place.

Estoria provides APIs for saving and loading aggregates as streams of events that represent changes to the aggregate's (and thus its underlying root entity's) state.

### Aggregate Store

An aggregate store is a component that can save and load aggregates.

Estoria provides a core event-sourced aggregate store that uses an underlying event store to save and load aggregates. It also provides composable aggregate store layers that provide additional capabilities, such as aggregate snapshotting, caching, and the ability to inject lifecycle hooks.

### Event Store

An event store is a component that can save and load events.

Estoria provides [officially-supported event store implementations](../component_library/#event-store-implementations) for a variety of storage backends from AWS, Azure, and Google Cloud.

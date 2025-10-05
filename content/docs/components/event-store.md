---
title: Event Store
type: docs
prev: docs/getting-started/
next: docs/components/aggregate_store
sidebar:
  open: true
weight: 110
---

An **event** is something that has occurred in the past, often with associated information.

An **event stream** is a uniquely-identified, ordered sequence of events.

An **event store** reads events from and appends events to event streams.

In an event-sourced system, the event store forms the foundation of the persistence layer. By maintaining a an append-only record of all changes to entities, it stores the complete history of their state over time.

## Event Store Implementations

Select an event store from [Event Store Implementations](../../component-library/#event-store-implementations) in the component library, or build your own. Anything implementing the following interface can be used as an event store with Estoria:

```go
import (
  "github.com/go-estoria/estoria/eventstore"
  "github.com/go-estoria/estoria/typeid"
)

interface {
    AppendStream(context.Context, typeid.ID, []*eventstore.WritableEvent, eventstore.AppendStreamOptions) error
    ReadStream(context.Context, typeid.ID, eventstore.ReadStreamOptions) (eventstore.StreamIterator, error)
}
```

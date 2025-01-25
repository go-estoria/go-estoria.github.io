---
title: Event Store
type: docs
prev: docs/getting_started/
next: docs/core_components/aggregate_store
sidebar:
  open: true
weight: 110
---

An **event store** loads and stores events.

An **event** is a collection of data associated with a particular event stream.

An **event stream** is an ordered collection of events associated with a specific event stream ID.

Event stores operate on event streams, which are sequences of events associated with a particular stream ID. Stream vectors are used to describe specific queries on event streams.

## Overview

Anything implementing the following interface can be used as an event store with Estoria:

```go
import (
  "github.com/go-estoria/estoria/eventstore"
  "github.com/go-estoria/estoria/typeid"
)

interface {
    AppendStream(context.Context, typeid.UUID, []*eventstore.WritableEvent, eventstore.AppendStreamOptions) error

    ReadStream(context.Context, typeid.UUID, eventstore.ReadStreamOptions) (eventstore.StreamIterator, error)
}
```

## Appending Events

## Reading Events

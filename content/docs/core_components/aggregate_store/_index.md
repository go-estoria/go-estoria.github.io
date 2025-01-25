---
title: Aggregate Store
type: docs
prev: docs/getting_started/
next: docs/core_components/aggregate_store
sidebar:
  open: true
weight: 210
---

An aggregate store loads and saves aggregates.

## Overview

Anything implementing the following interface can be used as an aggregate store with Estoria:

```go
import (
  "github.com/go-estoria/estoria/aggregatestore"
  "github.com/go-estoria/estoria/typeid"
)

interface {
    New(id uuid.UUID) (*aggregatestore.Aggregate[E], error)
    Load(context.Context, typeid.UUID, aggregatestore.LoadOptions) (*Aggregate[E], error)
    Hydrate(context.Context, *aggregatestore.Aggregate[E], aggregatestore.HydrateOptions) error
    Save(context.Context, *aggregatestore.Aggregate[E], aggregatestore.SaveOptions) error
}
```

## Appending Events

## Reading Events

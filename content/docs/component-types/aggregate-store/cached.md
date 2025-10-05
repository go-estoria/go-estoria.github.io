---
title: Cached Aggregate Store
type: docs
weight: 230
---

A cached aggregate store wraps an existing aggregate store and loads and saves aggregates using a cache.

Aggregate caching reduces the frequency with which aggregates must be loaded from persistence. This can significantly improve performance, especially for workloads that involve frequent loading of aggregates, such as in stream processing applications.

## Usage

A cached aggregate store requires an inner aggregate store (to defer to when hydrating aggregates) and an aggregate cache (to load and save cached aggregates).

```go
import (
    "context"
    "github.com/go-estoria/estoria/aggregatestore"
    "github.com/go-estoria/estoria-contrib/bigcache/aggregatecache"
)

func main() {
    // create an aggregate store
    store, _ := aggregatestore.NewEventSourcedStore(eventStore, NewThing)

    // create an aggregate cache
    cache := aggregatecache.New()

    // wrap an aggregate store with caching capabilities
    cachedStore, _ := aggregatestore.NewCachedStore(store, cache)
}
```

## Aggregate Caches

While the cached aggregate store is a core Estoria component, it relies on an aggregate cache implementation to provide the actual caching functionality. See [Aggregate Cache Implementations](../../../component-library/#aggregate-cache-implementations) in the component library for more information on available aggregate cache implementations.

## Custom Cache Implementations

Anything implementing the following interface can be used as an aggregate cache with Estoria:

```go
type AggregateCache[E estoria.Entity] interface {
	GetAggregate(ctx context.Context, aggregateID uuid.UUID) (*aggregatestore.Aggregate[E], error)
	PutAggregate(ctx context.Context, aggregate *aggregatestore.Aggregate[E]) error
}
```

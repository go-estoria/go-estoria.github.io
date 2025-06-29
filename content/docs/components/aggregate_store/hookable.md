---
title: Hookable Aggregate Store
type: docs
prev: docs/components/event_store/
weight: 240
---

A hookable aggregate store wraps an existing aggregate store and provides lifecycle hooks for the store's operations.

Lifecycle hooks are functions that are called before and after aggregates are created, loaded, hydrated, and/or saved. They can be used for things such as logging, telemetry collection, or other side effects.

## Usage

A hookable aggregate store requires an inner aggregate store (to perform actual aggregate store operations) and zero or more configured lifecycle hooks.

```go
import (
    "context"
    "github.com/go-estoria/estoria/aggregatestore"
)

func main() {
    // create an aggregate store
    store, _ := aggregatestore.NewEventSourcedStore(eventStore, NewThing)

    // wrap an aggregate store with hook capabilities
    hookableStore, _ := aggregatestore.NewHookableStore(store)

    // define some lifecycle hooks
    hookableStore.BeforeLoad(func(ctx context.Context, id uuid.UUID) error {
		log.Printf("loading aggregate %s", id.String())
		return nil
	})
	hookableStore.AfterLoad(func(ctx context.Context, aggregate *aggregatestore.Aggregate[Account]) error {
		log.Printf("loaded aggregate %s at version %d", aggregate.ID(), aggregate.Version())
		return nil
	})
	hookableStore.BeforeSave(func(ctx context.Context, aggregate *aggregatestore.Aggregate[Account]) error {
		log.Printf("saving aggregate %s from version %d", aggregate.ID(), aggregate.Version())
		return nil
	})
	hookableStore.AfterSave(func(ctx context.Context, aggregate *aggregatestore.Aggregate[Account]) error {
		log.Printf("saved aggregate %s to version %d", aggregate.ID(), aggregate.Version())
		return nil
	})
}
```
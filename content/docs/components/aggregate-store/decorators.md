---
title: Custom Decorators
type: docs
weight: 240
---

A custom decorator wraps an existing aggregate store to add application-owned behavior around its operations.

Estoria deliberately ships no hook framework for this. A decorator you write yourself is a handful of lines, is transparent about exactly what runs and when, and leaves you owning the consequences for caching, snapshotting, and error semantics. Decoration is also how Estoria's own [snapshotting](../snapshotting) and [cached](../cached) stores are built: embed the inner store, override only the methods you care about, and everything else forwards automatically.

## Observing Saves

The most common case: react every time an aggregate is saved — push an update to connected clients, count a metric, write an audit line. Wrap `Save`, delegate first, and observe only after the inner store reports success:

```go
// broadcastingStore pushes every successfully saved board to all SSE clients.
type broadcastingStore struct {
    aggregatestore.Store[Board]
    hub *hub
}

func (s broadcastingStore) Save(ctx context.Context, aggregate *aggregatestore.Aggregate[Board], opts *aggregatestore.SaveOptions) error {
    if err := s.Store.Save(ctx, aggregate, opts); err != nil {
        return err
    }

    s.hub.broadcast(boardMessage{Version: aggregate.Version(), Board: aggregate.State()})

    return nil
}
```

Keep observers non-failing. By the time the observer runs, the save has durably succeeded; returning an error here would misreport a completed save as a failed one. The [Kanban, Chess, Orders, and Fleet examples](/docs/examples) each use exactly this decorator to power their live UIs.

## Injecting Ambient Metadata

To attach ambient context — correlation and causation IDs, the acting user, a trace — to every event a save persists, amend the queued events before delegating:

```go
// tracingStore stamps every saved event with the request's correlation ID.
type tracingStore struct {
    aggregatestore.Store[Order]
}

func (s tracingStore) Save(ctx context.Context, aggregate *aggregatestore.Aggregate[Order], opts *aggregatestore.SaveOptions) error {
    aggregate.MergeEventMetadata(map[string]string{"correlation_id": correlationIDFrom(ctx)})
    return s.Store.Save(ctx, aggregate, opts)
}
```

`MergeEventMetadata` merges into all of the aggregate's unsaved events, with the latest write per key winning.

## The Save-Outcome Contract

Estoria classifies every save failure by what reached the stream. An error from `Save` resolves, via `aggregatestore.SaveOutcome`, to one of three outcomes:

- **`AppendOutcomeAppended`** — the error carries `aggregatestore.ErrEventsAppended`: the events were durably appended before the failure, persisted state moved ahead of the in-memory aggregate, and the caller recovers by reloading, never by retrying the save.
- **`AppendOutcomeNothingAppended`** — it carries `aggregatestore.ErrNoEventsAppended`: this save wrote nothing.
- **`AppendOutcomeUnknown`** — it carries neither: a store can commit and lose its response, and only reading the stream resolves what happened.

Resolve outcomes with `SaveOutcome` rather than `errors.Is`: a wrapped cause can carry the opposite marker from the error that wraps it, and `errors.Is` searches the whole tree, so it can confirm both sentinels on the same error.

Every decorator in the stack must preserve that classification:

- Return the inner `Save` error unchanged, or wrap it with `%w`, so the markers survive.
- Mark the errors you originate: a failure raised before delegating inward appended nothing — wrap `ErrNoEventsAppended`; a failure raised after a successful inner save follows facts already in the stream — wrap `ErrEventsAppended`.
- Don't fail after a successful inner save for a side effect's sake (the observer rule above); a decorator that genuinely can fail there must carry `ErrEventsAppended`, because that is now the truth of the save.

## Side Effects That Must Not Be Lost

A decorator runs outside the event store's transaction: the process can die after the append and before the observer runs, and the side effect is simply gone. That is fine for a UI nudge and wrong for anything that must happen. Deliveries that must survive belong in the [transactional outbox](/docs/cqrs#the-transactional-outbox), where each pending delivery commits in the same database transaction as the events themselves.

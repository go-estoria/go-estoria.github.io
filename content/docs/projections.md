---
title: Projections
type: docs
prev: docs/components/aggregate-cache/
next: docs/telemetry/
weight: 800
---

A **projection** is a read model built by processing a stream of events. Projections create views of your data optimized for querying and reporting.

Estoria's projection toolkit lives in the `projection` package and its subpackages:

- `projection` — versioned projection identity and the one-shot `Fold`
- `projection/checkpointstore` — persistence for a projection's progress
- `projection/processor` — the continuous runtime: replay from a checkpoint, then tail

## Folding a Stream

For an on-demand view, obtain a stream iterator from your event store and fold it. The handler is called for each event in the stream and updates the view's state from the event data:

```go
import (
    "context"
    "encoding/json"

    "github.com/go-estoria/estoria/eventstore"
    "github.com/go-estoria/estoria/projection"
)

// get a stream iterator
iter, _ := eventStore.ReadStream(ctx, streamID, eventstore.ReadStreamOptions{})

// create the fold
fold, _ := projection.NewFold(iter)

// simple projection state
var balance int

// run the projection
_, err := fold.Project(ctx, projection.EventHandlerFunc(func(_ context.Context, event *eventstore.Event) error {
    if event.ID.Type != "balancechanged" {
        return nil
    }

    var e BalanceChangedEvent
    if err := json.Unmarshal(event.Data, &e); err != nil {
        return err
    }

    balance += e.Amount
    return nil
}))
```

Projections operate directly on event streams, not aggregates: the handler receives raw `*eventstore.Event`s and must determine event types and deserialize payloads itself. `Project` reads until the end of the stream and returns how many events were projected.

## Projecting All Streams

A fold over a single stream sees one aggregate's history. To build read models spanning every stream, use `ReadAll`, available from event stores that implement `eventstore.GlobalReader`:

```go
globalReader, ok := eventStore.(eventstore.GlobalReader)
if !ok {
    // this event store does not support global reads
}

iter, _ := globalReader.ReadAll(ctx, eventstore.ReadAllOptions{})

fold, _ := projection.NewFold(iter)
```

Events arrive in ascending global order, each carrying a `GlobalPosition`. A read resumed with `ReadAllOptions{AfterPosition: position}` continues with the first unprocessed event — the position is exclusive.

## Continuous Projections

A read model that serves queries needs to stay current, not be rebuilt per request. `projection/processor` runs a projection continuously: it replays history from the projection's checkpoint, drains to the head of the global event sequence, then tails it by polling.

A continuous projection is identified by a `projection.ID` — a name plus a 1-based version, rendered canonically as `account_balances_v1`. The ID keys the projection's checkpoint, and its string form is designed to serve as the storage suffix for versioned tables, indexes, or collections, so a new version of a read model can be built alongside the one still serving reads.

Progress persists through the `checkpointstore.Store` interface — `Load`, `Save`, and `Delete` of a global position per projection ID. The `checkpointstore/memory` store suits tests and single-process tools; for durable deployments, implement the interface over the database your read model lives in. The `checkpointstore/storetest` suite verifies an implementation against the contract.

```go
import (
    "github.com/go-estoria/estoria/projection"
    checkpointmemory "github.com/go-estoria/estoria/projection/checkpointstore/memory"
    "github.com/go-estoria/estoria/projection/processor"
)

id := projection.ID{Name: "account_balances", Version: 1}

handler := projection.EventHandlerFunc(func(_ context.Context, event *eventstore.Event) error {
    if event.ID.Type != "balancechanged" {
        return nil
    }

    var e BalanceChangedEvent
    if err := json.Unmarshal(event.Data, &e); err != nil {
        return err
    }

    // The position guard makes redelivery harmless: an event at or below
    // the row's high-water mark has already been applied.
    _, err := db.Exec(
        `UPDATE account_balances_v1
            SET balance = balance + $1, last_position = $2
          WHERE account_id = $3 AND last_position < $2`,
        e.Amount, *event.GlobalPosition, event.StreamID.UUID.String(),
    )
    return err
})

proc, err := processor.New(globalReader, checkpointmemory.NewCheckpointStore(), id, handler)
if err != nil {
    // ...
}

// Run blocks: replay from the checkpoint, drain to the head, then tail by
// polling. Cancel the context to stop; a new Processor resumes from the
// checkpoint.
err = proc.Run(ctx)
```

`CaughtUp` exposes a channel that closes when the replay first reaches the head of the event sequence — after the head position has been durably checkpointed. While tailing, the processor re-saves its unchanged position every idle poll cycle, so checkpoint recency doubles as a liveness signal: a checkpoint much older than the poll interval means the processor is not running.

Polling cadence, read batch sizes, and checkpoint frequency are configurable via `WithPollInterval`, `WithBatchSize`, and `WithCheckpointEvery`.

## Delivery Guarantees

Delivery is **at least once**: the handler runs before the checkpoint is saved, so a crash between the two redelivers the event on restart. Two consequences for handler design:

- **Handlers must be idempotent.** Applying the same event twice must leave the read model unchanged — the position guard in the example above is the standard pattern.
- **Handlers should not perform external side effects.** Sending an email or calling a webhook from a projection handler repeats the side effect on every redelivery — and every rebuild of a versioned projection replays all of history, so "am I replaying?" is not a meaningful question. Deliver side effects through the [transactional outbox](../cqrs#the-transactional-outbox) instead.

## Checkpoints Are Not Snapshots

The two are easy to conflate — in some ecosystems "checkpoint" *means* a state snapshot — but in Estoria they are different things:

- A **checkpoint** is a saved *location*: the global position a projection has processed through. Restoring from one means replaying events after it.
- A **snapshot** is saved *state*: an aggregate's materialized form at a stream version, used by the [snapshot store](../components/snapshot-store) to shortcut aggregate loading. Restoring from one means deserializing it.

A projection's state lives in its read model storage; Estoria persists only the checkpoint.

## Error Handling

By default, a handler error stops the projection and is returned. For a `Fold`, that ends the run. For the processor, the failed event remains ahead of the checkpoint, so a restart redelivers it — a poison event blocks the projection rather than being silently skipped.

Both accept `WithContinueOnHandlerError(true)` to log the error and continue instead. The processor then checkpoints past the failed event with the rest of its batch, so it is **not** redelivered on restart.

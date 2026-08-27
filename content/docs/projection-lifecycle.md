---
title: Projection Lifecycle
type: docs
prev: docs/projections/
next: docs/telemetry/
weight: 805
---

Read models change: a bug in a handler, a new column derived from history, a better storage layout. The `projection/lifecycle` package makes rebuilding one a first-class, durable operation — a new version is built alongside the version still serving reads, promoted with a recorded cutover, and rolled back or retired the same way. Nothing stops serving while any of it happens.

The package builds on [versioned projection identity](../projections#continuous-projections): each rebuild targets a fresh `projection.ID` — `account_balances_v2` beside the live `account_balances_v1` — with its own storage and its own checkpoint. Version numbers are never reused, so an abandoned or rolled-back build's residue belongs to a permanently dead identity and cannot leak into any later build.

## The Lifecycle Is Itself Event-Sourced

Each named projection has one lifecycle: an ordinary Estoria aggregate whose stream records every consequential decision — initiating a rebuild, claiming it, catching up, promoting, rolling back, abandoning, retiring. The projection's name is the arbitration domain: competing decisions append to the same stream under optimistic concurrency, and exactly one wins. The loser's command returns a version-mismatch error, and reloading observes the transition that won.

An `Orchestrator` mediates it all:

```go
import (
    "github.com/go-estoria/estoria/projection"
    "github.com/go-estoria/estoria/projection/lifecycle"
)

orchestrator, err := lifecycle.NewOrchestrator(lifecycle.Config{
    Events:          eventStore,   // the domain events, via global reads
    Checkpoints:     checkpoints,  // progress for each version being built
    Handler:         newHandler,   // handler factory, per versioned ID
    LifecycleEvents: eventStore,   // where lifecycle streams live
})
```

`Handler` is a factory — `func(projection.ID) (projection.EventHandler, error)` — because the versioned ID flows in: each version's handler targets its own table or collection. A handler that can remove its version's storage implements `projection.Teardowner`, which retirement requires. The factory must not prepare or validate storage: retirement repair re-resolves handlers for versions whose storage is already gone.

## Rebuilding

`Begin` records the rebuild and returns a handle; `Run` drives it:

```go
rebuild, err := orchestrator.Begin(ctx, "account_balances", "backfill activity stats")
if err != nil {
    // a rebuild is already in flight, or the state cannot be loaded
}

go func() {
    err := rebuild.Run(ctx) // blocks: claim, build, catch up, certify, tail
}()
```

`Run` durably claims the attempt for this process, starts a processor for the target version, replays history, and — on reaching the head — records the catch-up and certifies it in-process. It then keeps tailing until the context is canceled or the attempt ends. While it runs, a reconcile loop rehydrates the lifecycle on an interval, so a builder superseded elsewhere winds itself down instead of running until an operator notices.

The attempt moves through phases — `created`, `building`, `caught_up`, `promoted`, `retiring` — and the handle's commands move it:

- **`Promote`** cuts reads over to the target version by recording the cutover. It requires the current catch-up certification: only the run that drained this rebuild to the head may promote it — a handle that merely loaded a caught-up rebuild gets `ErrNotCertified` and must `Run` (re-certify) first. With `WithAutoPromote(true)`, the run promotes itself on catching up.
- **`Rollback`** (after promotion) reverts reads to the previous version and ends the attempt.
- **`Abandon`** (before promotion) gives up and ends the attempt.
- **`Retire`** (after promotion) destroys the previous version's storage and completes the rebuild — see [Retirement](#retirement).

A rolled-back or abandoned version's table and checkpoint are deliberately left in place, inert: its version number is never reused, and nothing can prove no concurrent builder still holds a handle to it. Collect the residue explicitly, on your schedule.

## Cutover

Promotion records an event; it does not reach into your infrastructure. Convergence is the cutover worker's job: it folds the recorded cutover history and applies each projection's current cutover through every registered `CutoverSetter`, then tails the global sequence and delivers each new flip.

```go
router := lifecycle.NewMemoryRouter()

worker, err := lifecycle.NewWorker(eventStore, lifecycle.WithCutoverSetter(router))

go worker.Run(ctx)
<-worker.Ready() // the router now holds the recorded routing truth
```

The `Router` interface answers `Live(ctx, name)` — which version serves reads. In a **logical cutover**, the read path consults it per query and composes the versioned storage name; in a **physical cutover**, a setter repoints a database view or alias and readers never consult anything. `MemoryRouter` is an in-process router-and-setter for the worker to converge; `StreamRouter` reads the lifecycle streams directly on a refresh interval, for processes that want routing without running a worker.

Workers are stateless — any number may run concurrently, and a restarted worker refolds from zero — so every process in a fleet learns about flips the same way.

## Claims, Crashes, and Takeover

Nothing fences data-plane writes within one attempt: two processors building the same target version would interleave writes into one table and one checkpoint. So `Run` records a durable claim before its processor exists, and a second `Run` finding that claim standing refuses with `ErrClaimStanding` rather than joining in.

A graceful wind-down releases its claim durably on the way out, and a successor resumes transparently:

```go
rebuild, err := orchestrator.Resume(ctx, "account_balances")

err = rebuild.Run(ctx) // admitted: the previous run released its claim
```

A crashed process releases nothing. Its claim stands until an operator — who alone can vouch that the process is gone — takes it over, with the attestation recorded durably in the claim that wins:

```go
err = rebuild.Run(ctx, lifecycle.WithTakeover("j.linse", "builder pod OOM-killed, confirmed gone"))
```

The release happens during `Run`'s wind-down, so a shutting-down process must wait for `Run` to return before exiting — a process that exits early leaves a standing claim behind, and the next run needs a takeover exactly as after a crash.

## Retirement

Retiring destroys the previous version's storage, so it is the one gated transition. The gate is a durable **retirement policy**: the witness IDs that must attest — or the explicit, audited choice to retire unwitnessed — recorded on the lifecycle stream, where a restarted process configured with fewer witnesses cannot silently weaken it.

```go
err := orchestrator.SetRetirementPolicy(ctx, "account_balances", lifecycle.RetirementPolicyChange{
    Witnesses: []string{"router"},
    Actor:     "j.linse",
    Reason:    "gate retirements on read convergence",
})
```

A witness attests that it serves the exact live (version, revision) pair — typically a router vouching that no governed read path still resolves to the version about to be destroyed. `Retire` collects attestations while rollback is still possible, records the reservation (which forfeits rollback), re-attests, tears down the storage through the handler's `Teardowner`, deletes the checkpoint, and records completion. A retirement interrupted partway is repaired by calling `Retire` again. A projection with no recorded policy refuses to retire unless the call carries an audited `WithRetirementOverride(actor, reason)`.

A first rebuild has no previous version: `Retire` completes it trivially, destroying nothing.

## Steady-State Serving

Steady-state processing of the live version is deliberately not a lifecycle concern — it is a plain [`processor.Processor`](../projections#continuous-projections). The one operational rule: while a rebuild runs, the lifecycle's own processor tails the target version, and from promotion until that run winds down, the target *is* the live version — so run a steady-state processor for the live version unless the in-flight attempt targets it. The previous version's processor must be stopped before `Retire`.

The [Ledger example](../examples) wires all of this into a runnable service — versioned Postgres tables, a durable checkpoint store, the serving hand-off, and a console driving every command above.

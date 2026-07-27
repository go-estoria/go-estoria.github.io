---
title: Examples
type: docs
prev: docs/typeids/
weight: 840
---

Every example below lives in
[estoria-examples](https://github.com/go-estoria/estoria-examples) as a
self-contained Go module pinning released versions of `estoria` and
`estoria-contrib`. Any one of them can be copied out of the repo on its own
and run.

## Applications

Complete, runnable apps with web UIs, showing what event sourcing with Estoria
looks like end to end.

{{< cards >}}
  {{< card link="https://github.com/go-estoria/estoria-examples/tree/main/kanban" title="Kanban" icon="view-boards" subtitle="A real-time collaborative board with a time-travel slider, live sync over SSE, optimistic concurrency surfaced in the UI, and snapshotting. SQLite — no Docker required." >}}
  {{< card link="https://github.com/go-estoria/estoria-examples/tree/main/orders" title="Orders" icon="shopping-cart" subtitle="An order-fulfillment service: a strict state-machine domain, the transactional outbox delivering events to a CQRS read model and webhook log, and a live admin dashboard. Postgres." >}}
  {{< card link="https://github.com/go-estoria/estoria-examples/tree/main/fleet" title="Fleet" icon="chip" subtitle="An IoT sensor-fleet dashboard with an in-process device simulator: long streams, the full snapshotting and caching decorator stack, and a live hydration benchmark. SQLite." >}}
  {{< card link="https://github.com/go-estoria/estoria-examples/tree/main/chess" title="Chess" icon="puzzle" subtitle="Live two-player chess where each game is an event stream: move legality enforced inside ApplyTo, a replay slider, optimistic concurrency as turn-race protection, and PGN export. SQLite." >}}
  {{< card link="https://github.com/go-estoria/estoria-examples/tree/main/inspector" title="Inspector" icon="search" subtitle="A read-only web tool for browsing any supported event store: stream lists, event paging, payload inspection, and a live global-feed tail." >}}
{{< /cards >}}

### Try them without installing anything

Two are running live. Both are seeded fresh at the top of every hour, so
anything you do to them is temporary by design — and both accept writes from
anyone, which is exactly what makes optimistic concurrency worth watching.

- **[chess.demo.estoria.dev](https://chess.demo.estoria.dev)** — start a game,
  then open it in a second tab and play both sides. Every move is an event
  appended to that game's stream; the replay slider re-derives the position
  from any prefix of it.
- **[orders.demo.estoria.dev](https://orders.demo.estoria.dev)** — place an
  order and watch the outbox monitor. The order list doesn't move when the
  command succeeds; it moves a moment later, when the outbox processor
  delivers the event and the read model catches up. That gap is the CQRS split,
  made visible.

These are a convenience, not part of the project's infrastructure. Every
example runs locally with one command, and the deployment recipe
(`Dockerfile` and `railway.toml`) lives beside each one in the repository.

### Where to look for a given concept

| If you want to see… | Read | In the code |
| --- | --- | --- |
| Entities, events, and `ApplyTo` | [Defining Entities](/docs/getting-started/defining-entities), [Defining Events](/docs/getting-started/defining-events) | `kanban/board.go`, `kanban/board_events.go` |
| Optimistic concurrency and version conflicts | [Aggregate Store](/docs/components/aggregate-store) | `kanban/server.go` (`runCommand`), `chess/server.go` |
| Loading an aggregate at a past version | [Event-Sourced Aggregate Store](/docs/components/aggregate-store/event-sourced) | `kanban/server.go`, `chess/server.go` |
| Snapshotting on a policy | [Snapshotting Aggregate Store](/docs/components/aggregate-store/snapshotting), [Snapshot Store](/docs/components/snapshot-store) | `kanban/main.go`, `fleet/main.go` |
| Hooks reacting to a save | [Hookable Aggregate Store](/docs/components/aggregate-store/hookable) | `kanban/main.go` (SSE broadcast on `AfterSave`) |
| Aggregate caching, and what it's worth | [Cached Aggregate Store](/docs/components/aggregate-store/cached), [Aggregate Cache](/docs/components/aggregate-cache) | `fleet/main.go`, plus the hydration benchmark in `fleet/server.go` |
| Projections over a stream | [Projections](/docs/projections) | `kanban/server.go` (activity feed) |
| CQRS with a read model | [CQRS](/docs/cqrs) | `orders/readmodel.go` |
| The transactional outbox | [CQRS](/docs/cqrs) | `orders/main.go`, `orders/readmodel.go` |
| Reading an event store directly | [Event Store](/docs/components/event-store) | `inspector/backend.go` |

## Backend quickstarts

The same short walkthrough of Estoria's core components, each wired to a
different storage backend. Every one ships a `docker-compose.yml` and a
Makefile for its dependencies.

| Quickstart | What it covers |
| --- | --- |
| [PostgreSQL](https://github.com/go-estoria/estoria-examples/tree/main/postgres) | Postgres event store, including the transactional outbox for reliable event delivery to external consumers. |
| [MongoDB](https://github.com/go-estoria/estoria-examples/tree/main/mongodb) | MongoDB event store using a single-collection strategy. |
| [KurrentDB](https://github.com/go-estoria/estoria-examples/tree/main/kurrent) | KurrentDB (formerly EventStoreDB) event store with native stream mapping. |
| [OpenTelemetry](https://github.com/go-estoria/estoria-examples/tree/main/opentelemetry) | Instrumenting event, aggregate, and snapshot stores with OTEL tracing and metrics, against Jaeger, Prometheus, and Grafana. See [Telemetry](/docs/telemetry). |

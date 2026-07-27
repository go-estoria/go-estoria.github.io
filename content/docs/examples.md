---
title: Examples
type: docs
prev: docs/getting-started/
next: docs/components/
weight: 130
---

Every example below lives in
[estoria-examples](https://github.com/go-estoria/estoria-examples) as a
self-contained Go module pinning released versions of `estoria` and
`estoria-contrib`. Any one of them can be copied out of the repo on its own
and run.

## Backend Quickstarts

| Quickstart | What it covers |
| --- | --- |
| [PostgreSQL](https://github.com/go-estoria/estoria-examples/tree/main/postgres) | Postgres event store, including the transactional outbox for reliable event delivery to external consumers. |
| [MongoDB](https://github.com/go-estoria/estoria-examples/tree/main/mongodb) | MongoDB event store using a single-collection strategy. |
| [KurrentDB](https://github.com/go-estoria/estoria-examples/tree/main/kurrent) | KurrentDB (formerly EventStoreDB) event store with native stream mapping. |
| [OpenTelemetry](https://github.com/go-estoria/estoria-examples/tree/main/opentelemetry) | Instrumenting event, aggregate, and snapshot stores with OTEL tracing and metrics, against Jaeger, Prometheus, and Grafana. See [Telemetry](/docs/telemetry). |

## Applications

Complete, runnable apps with web UIs, showing what event sourcing with Estoria
looks like end to end.

| Example | What it demonstrates | Try it |
| --- | --- | --- |
| [Kanban](https://github.com/go-estoria/estoria-examples/tree/main/kanban) | A real-time collaborative board with a time-travel slider, live sync over SSE, optimistic concurrency surfaced in the UI, and snapshotting. SQLite — no Docker required. | |
| [Orders](https://github.com/go-estoria/estoria-examples/tree/main/orders) | An order-fulfillment service: a strict state-machine domain, the transactional outbox delivering events to a CQRS read model and webhook log, and a live admin dashboard. Postgres. | [orders.demo.estoria.dev](https://orders.demo.estoria.dev) |
| [Fleet](https://github.com/go-estoria/estoria-examples/tree/main/fleet) | An IoT sensor-fleet dashboard with an in-process device simulator: long streams, the full snapshotting and caching decorator stack, and a live hydration benchmark. SQLite. | |
| [Chess](https://github.com/go-estoria/estoria-examples/tree/main/chess) | Two-player chess where each game is an event stream: move legality enforced inside `ApplyTo`, a replay slider, optimistic concurrency as turn-race protection, and PGN export. SQLite. | [chess.demo.estoria.dev](https://chess.demo.estoria.dev) |
| [Inspector](https://github.com/go-estoria/estoria-examples/tree/main/inspector) | A read-only web tool for browsing any supported event store: stream lists, event paging, payload inspection, and a live global-feed tail. | |

The hosted demos are seeded fresh at the top of every hour and accept writes
from anyone — which is what makes the concurrency behavior worth watching.

### Concepts

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

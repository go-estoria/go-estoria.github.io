---
title: Snapshot Store
type: docs
prev: docs/getting-started/
next: docs/components/aggregate-store
sidebar:
  open: false
weight: 260
---

A **snapshot** is a captured representation of an aggregate's state at a specific point in time. It consists of the aggregate's ID, current version, and serialized entity state.

A **snapshot store** loads and saves snapshots.

The [snapshotting aggregate store](../aggregate-store/snapshotting) uses a snapshot store to reduce the number of events that need to be loaded and replayed when hydrating an aggregate.

## Implementations

Core ships an [in-memory store](/docs/components/snapshot-store/in_memory/) for tests and prototypes, and an [event stream store](/docs/components/snapshot-store/event_stream/) that keeps snapshots as events in your event store. For PostgreSQL, estoria-contrib ships a [snapshot store](/docs/integrations/#snapshot-stores) that keeps them in a table of their own. See [Snapshot Store Implementations](/component-library/#snapshot-stores) in the component library for more information on available snapshot store implementations.
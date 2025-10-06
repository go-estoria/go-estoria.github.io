---
title: Snapshot Store
type: docs
prev: docs/getting-started/
next: docs/components/aggregate_store
sidebar:
  open: false
weight: 260
---

A s**napshot** is a captured representation of an aggregate's state at a specific point in time. It consists of the aggregate's ID, current version, and serialized entity state.

A **snapshot store** loads and saves snapshots.

The [snapshotting aggregate store](../aggregate-store/snapshotting) uses a snapshot store to reduce the number of events that need to be loaded and replayed when hydrating an aggregate.

## Implementations

See [Snapshot Store Implementations](/component-library/#snapshot-stores) in the component library for more information on available aggregate cache implementations.
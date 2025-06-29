---
title: Snapshot Store
type: docs
prev: docs/getting_started/
next: docs/components/aggregate_store
sidebar:
  open: true
weight: 260
---

A snapshot store loads and saves snapshots of aggregates.

A snapshot consists of the aggregate's ID, current version, and serialized entity state. The [snapshotting aggregate store](../aggregate_store/snapshotting) uses a snapshot store to reduce the number of events that need to be loaded and replayed when hydrating an aggregate.

## Implementations

See [Snapshot Store Implementations](../../component_library/#snapshot-store-implementations) in the component library for more information on available aggregate cache implementations.
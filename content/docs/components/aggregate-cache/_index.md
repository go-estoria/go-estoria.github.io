---
title: Aggregate Cache
type: docs
prev: docs/components/snapshot-store/
next: docs/projections/
sidebar:
  open: false
weight: 270
---

An **aggregate cache** loads and saves cached aggregate data.

The [cached aggregate store](../aggregate-store/cached) uses an aggregate cache to reduce the number of events that need to be loaded and replayed when hydrating an aggregate.

## Implementations

See [Aggregate Cache Implementations](/component-library/#aggregate-caches) in the component library for more information on available aggregate cache implementations.

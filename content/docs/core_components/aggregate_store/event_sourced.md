---
title: Event-Sourced Aggregate Store
type: docs
prev: docs/core_components/
weight: 220
---

The event-sourced aggregate store loads and saves aggregates using an event store.

Aggregates are hydrated by fetching events from the event store and applying them to the aggregate to reconstruct its state. Aggregates are saved by appending events representing incremental state changes to the event store.

This functionality -- the loading and saving of aggregates using event streams -- is central to event-sourcing using Estoria.

## Overview

An event-sourced aggregate store is scoped to a specific entity type. It thus knows how to load and store specific types of events that are applicable to that entity type.

## Behavior

### New

When creating a new aggregate, the event-sourced aggregate store creates a new aggregate instance and assigns a default version of 0 (no applied events) and default entity using the provided entity factory.

### Load

When loading an aggregate, the event-sourced store queries its event store for all events matching the aggregate ID and applies these events in-order by aggregate version to the aggregate's entity.

This behavior can be controller on a per-call basis by specifying `LoadOptions` with the call.

### Hydrate

When hydrating an aggregate, the event-sourced store queries its event store for all events after the aggregate's current version and applies these events in-order by aggregate version to the aggregate's entity.

### Save

When saving an aggregate, the event-sourced store appends events to the event store and then applies the events to the aggregate's entity.

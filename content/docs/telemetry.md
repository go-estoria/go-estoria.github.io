---
title: Telemetry
type: docs
prev: docs/projections/
next: docs/cqrs/
weight: 810
---

## OpenTelemetry

Estoria components can be instrumented with [OpenTelemetry](https://opentelemetry.io/) to provide tracing and metrics. The [estoria-contrib](https://github.com/go-estoria/estoria-contrib/tree/main/opentelemetry) repository provides OTEL wrappers for event stores, aggregate stores, and snapshot stores. A complete runnable example can be found in [estoria-examples](https://github.com/go-estoria/estoria-examples/tree/main/opentelemetry).

```go
import (
    otelaggregatestore "github.com/go-estoria/estoria-contrib/opentelemetry/aggregatestore"
    oteleventstore "github.com/go-estoria/estoria-contrib/opentelemetry/eventstore"
    otelsnapshotstore "github.com/go-estoria/estoria-contrib/opentelemetry/snapshotstore"
)

// instrument an event store
eventStore, _err_ = oteleventstore.NewInstrumentedStore(eventStore)

// instrument an aggregate store
aggregateStore, _err_ = otelaggregatestore.NewInstrumentedStore(aggregateStore)

// instrument a snapshot store
snapshotStore, _err_ = otelsnapshotstore.NewInstrumentedStore(snapshotStore)
```

## Other Telemetry Providers

| Telemetry Provider | Driver | Components Supported | Details |
|--------------------|--------|----------------------|---------|
| [DataDog](https://github.com/go-estoria/estoria-contrib/tree/main/datadog) | https://github.com/DataDog/dd-trace-go | Event Store, Aggregate Store, Snapshot Store | Uses DataDog APM for tracing and metrics. |

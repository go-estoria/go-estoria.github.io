---
title: Event Sourcing for Go
toc: true
---

## What is Estoria?

Estoria is an event sourcing toolkit for Go. Event sourcing is a technique by which database entities are stored as a series of state-changing events. These events are then replayed to reconstruct an entity's current (or any previous) state.

Event sourcing is useful for any application that wishes to capture not just the current state of its domain data, but also the history of changes that led to that state.

{{< cards >}}
  {{< card link="/docs/getting-started" title="Getting Started" icon="sun" >}}
  {{< card link="/docs/component-types" title="Documentation" icon="book-open" >}}
  {{< card link="https://github.com/go-estoria/estoria-examples" title="Examples" icon="clipboard" >}}
  {{< card link="component-library" title="Component Library" icon="cog" >}}
  {{< card link="https://pkg.go.dev/github.com/go-estoria/estoria" title="GoDoc" icon="book-open" >}}
  {{< card link="https://github.com/go-estoria/estoria" title="GitHub" icon="github" >}}
{{< /cards >}}

---
title: Defining Entities
type: docs
prev: getting-started
next: getting-started/defining-events
weight: 121
---

An **entity** is a uniquely-identifyable domain object that we want to store as an event-sourced record. In Estoria, an entity is any Go type — no interface is required. The aggregate that manages an entity carries the identity and version; the entity itself only carries state.

Let's create a User entity type for our application:

```go
type User struct {
	ID   uuid.UUID
	Name string
}
```

The `ID` field here is optional — Estoria addresses aggregates by an ID it composes for you, so your entity may record its UUID, but it doesn't have to.

We must define a **factory function** for creating Users. Name it whatever you like; it should take a UUID and return a User object. The aggregate store uses it to build a fresh User before replaying events onto it. Here, we're assigning the ID to the User and setting a default name:

```go
func NewUser(id uuid.UUID) User {
	return User{ID: id, Name: "Unknown"}
}
```

Now that we've defined our User type, let's define some events that represent changes to a User's state.

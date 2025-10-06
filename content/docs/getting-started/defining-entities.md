---
title: Defining Entities
type: docs
prev: getting-started
next: getting-started/defining-events
weight: 121
---

An **entity** is a uniquely-identifyable domain object that we want to store as an event-sourced record.

Let's create a User entity type for our application. Each user will have a unique identifier, a name, and a timestamp for when the user was last updated.

```go
type User struct {
	ID        uuid.UUID
	Name      string
}
```

Our User type must implement the `estoria.Entity` interface, which requires defining an `EntityID()` method. This method should return a the User's _TypeID_, which is the User's UUID with an associated type name.

```go
func (a User) EntityID() typeid.ID {
	return typeid.New("user", a.ID)
}
```

Finally, we must define a **factory function** for creating Users. Name it whatever you like, and it should take a UUID and return a User object. Here, we're simply assigning the ID to the User and setting a default name.

```go
func NewUser(id uuid.UUID) User {
	return User{ID: id, Name: "Unknown"}
}
```

Now that we've defined our User type, let's define some events that represent changes to a User's state.

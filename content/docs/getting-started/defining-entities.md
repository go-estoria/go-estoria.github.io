---
title: Defining Entities
type: docs
prev: getting-started
next: getting-started/defining-events
weight: 121
---

An entity is a domain object type that we're interested in storing as an event-sourced record. Let's create a User entity type for our application:

```go
type User struct {
	ID        uuid.UUID
	Name      string
	UpdatedAt time.Time
}
```

Next, let's create a factory function for creating Users. It should take a UUID and return a new User object. Here, we're simply assigning the ID to the User and setting default values for Name and UpdatedAt.

```go
func NewUser(id uuid.UUID) User {
	return User{
		ID:        id,
		Name:      "Unknown",
		UpdatedAt: time.Now(),
	}
}
```

To work with Estoria, our User type must conform to the `estoria.Entity` interface, which means implementing `EntityID()` for our type. This method should return a the User's _TypeID_, which is the User's UUID with an associated type name.

```go
func (a User) EntityID() typeid.UUID {
	return typeid.FromUUID("user", a.ID)
}
```

Now that we've defined our User type, let's define some events that represent changes to a User's state.

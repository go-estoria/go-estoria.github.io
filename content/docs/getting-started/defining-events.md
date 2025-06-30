---
title: Defining Events
type: docs
prev: getting-started/defining-entities
next: getting-started/creating-an-event-store
weight: 122
---

An event represents something that has happened to an entity, often with associated information. For example, a User may choose to change their name. Let's define an entity event that represents this change to a User:

```go
type UserNameChanged struct {
	NewName string
}
```

Our event must implement the `estoria.EntityEvent[User]` interface, which requires three methods:

```go
// EventType returns a string representation of the event type's name.
func (UserNameChangedEvent) EventType() string { return "namechanged" }

// New is a factory function for creating new instances of the event.
func (UserNameChangedEvent) New() estoria.EntityEvent[User] { return &UserNameChangedEvent{} }

// ApplyTo defines the logic for how the event mutates the entity.
func (e UserNameChangedEvent) ApplyTo(_ context.Context, user User) (User, error) {
	log.Printf("applying user name changed event: %s -> %s", user.Name, e.NewName)
	user.Name = e.NewName
	user.UpdatedAt = time.Now()
	return user, nil
}
```

Now that we've defined an entity and some events, we're ready to create an event store for persisting events.

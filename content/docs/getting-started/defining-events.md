---
title: Defining Events
type: docs
prev: getting-started/defining-entities
next: getting-started/creating-an-event-store
weight: 122
---

An **event** represents a change to an entity, often with associated information. For example, a User changed their name. Events always represent something that has happened in the past. Let's define an event that represents this change to a User:

```go
type UserNameChanged struct {
	NewName string
}
```

Our event must implement the `estoria.DomainEvent[User]` interface:

`EventType()` returns a string representation of the event type's name. This is used to identify the event type when storing and retrieving events:

```go
func (UserNameChanged) EventType() string { return "namechanged" }
```

`New()` is a factory function for creating new instances of the event. This is used by Estoria when reading events from the event store to create new instances of the event type.

```go
import "github.com/go-estoria/estoria"

func (UserNameChanged) New() estoria.DomainEvent[User] {
	return &UserNameChanged{NewName: "Unknown User"}
}
```

Note that the `New()` method does not utilize its receiver, as it is a factory function that creates a new instance of the event. This enables Estoria to create new instances of the event when reading from the event store. It also gives us the ability to set default values for the event's fields.

`ApplyTo()` defines the logic for how the event produces the entity's next state. In this case, it sets the User's Name field based on the event's data:

```go
func (e UserNameChanged) ApplyTo(user User) User {
	user.Name = e.NewName
	return user
}
```

`ApplyTo` cannot fail: by the time an event is applied, it has already been persisted, and a persisted event is a fact. Validate commands *before* appending events to an aggregate — anything that could make the change invalid belongs in your command-handling code, not in `ApplyTo`. A payload that can't be decoded surfaces as an error during loading, before `ApplyTo` is ever reached.

Now that we've defined an entity and some events, we're ready to create an event store for persisting events.

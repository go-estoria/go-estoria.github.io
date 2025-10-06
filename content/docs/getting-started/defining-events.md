---
title: Defining Events
type: docs
prev: getting-started/defining-entities
next: getting-started/creating-an-event-store
weight: 122
---

An **event** represents a change to an entity, often with associated information. For example, an User changed their name. Events always represent something that has happened in the past. Let's define an event that represent this change to a User:

```go
type UserNameChanged struct {
	NewName string
}
```

Our event must implement the `estoria.EntityEvent[User]` interface:

`EventType()` returns a string representation of the event type's name. This is used to identify the event type when storing and retrieving events:

```go
func (UserNameChanged) EventType() string { return "namechanged" }
```

`New()` is a factory function for creating new instances of the event. This is used by Estoria when reading events from the event store to create new instances of the event type.

```go
import "github.com/go-estoria/estoria"

func (UserNameChanged) New() estoria.EntityEvent[User] {
	return &UserNameChanged{NewName: "Unknown User"}
}
```

Note that the `New()` method does not utilize its receiver, as it is a factory function that creates a new instance of the event. This enables Estoria to create new instances of the event when reading from the event store. It also gives us the ability to set default values for the event's fields.

`ApplyTo()` defines the logic for how the event mutates the entity. In this case, it sets the User's Name field based on the event's data.

```go
func (e UserNameChanged) ApplyTo(_ context.Context, user User) (User, error) {
	log.Printf("applying user name changed event: %s", e.NewName)
	user.Name = e.NewName
	user.UpdatedAt = time.Now()
	return user, nil
}
```

Now that we've defined an entity and some events, we're ready to create an event store for persisting events.

---
title: Defining Events
type: docs
prev: getting-started/defining-entities
next: getting-started/creating-an-event-store
weight: 122
---

An **event** represents a change to an entity, often with associated information. For example, a new User was created, or an existing User changed their name. Events always represent something that has happened in the past. Let's define two events that represent changes to a User:

```go
type UserCreated struct {
	Name string
}

type UserNameChanged struct {
	NewName string
}
```

Our events must implement the `estoria.EntityEvent[User]` interface, which requires three methods:

- `EventType()` returns a string representation of the event type's name.
- `New()` is a factory function for creating new instances of the event.
- `ApplyTo()` defines the logic for how the event mutates the entity.

Let's implement the `UserCreated` event first:

```go
func (UserCreatedEvent) EventType() string { return "created" }

func (UserCreatedEvent) New() estoria.EntityEvent[User] {
	return &UserCreatedEvent{Name: "Unknown User"}
}

func (e UserCreatedEvent) ApplyTo(_ context.Context, user User) (User, error) {
	log.Printf("applying user created event: %s", e.Name)
	user.Name = e.Name
	user.UpdatedAt = time.Now()
	return user, nil
}
```

Note that the `New()` method does not utilize its receiver, as it is a factory function that creates a new instance of the event. This enables Estoria to create new instances of the event when reading from the event store. It also gives us the ability to set default values for the event's fields.

`ApplyTo()` is where we define how the event mutates the User entity. In this case, it sets the User's Name and UpdatedAt fields based on the event's data.

Now let's implement the `UserNameChanged` event:

```go
type UserNameChanged struct {
	NewName string
}

// EventType returns a string representation of the event type's name.
func (UserNameChanged) EventType() string { return "namechanged" }

// New is a factory function for creating new instances of the event.
func (UserNameChanged) New() estoria.EntityEvent[User] {
	return &UserNameChanged{NewName: "Unknown User"}
}

// ApplyTo defines the logic for how the event mutates the entity.
func (e UserNameChanged) ApplyTo(_ context.Context, user User) (User, error) {
	log.Printf("applying user name changed event: %s", e.NewName)
	user.Name = e.NewName
	user.UpdatedAt = time.Now()
	return user, nil
}
```

Now that we've defined an entity and some events, we're ready to create an event store for persisting events.

---
title: Getting Started
type: docs
prev: docs/introduction
next: docs/getting-started/defining-entities
weight: 120
---

Estoria requires Go >=1.26.

```shell
go get github.com/go-estoria/estoria
```

>The core Estoria components only depend on the Go standard library and `github.com/gofrs/uuid/v5` for UUID support. Vendor-specific components are imported separately as needed from [estoria-contrib](https://github.com/go-estoria/estoria-contrib).

## Quickstart

Run all of the code below in a [Go playground](https://goplay.tools/snippet/sB6aDoAoNS_f). For complete applications built on Estoria — a collaborative kanban board, an order-fulfillment service, a sensor fleet, chess — see [Examples](/docs/examples).

### Entities

An entity is any type; no interface is required. Provide a factory function that builds one from a UUID:

```go
type User struct {
	ID   uuid.UUID
	Name string
}

func NewUser(id uuid.UUID) User {
	return User{ID: id, Name: "Unknown"}
}
```

### Events

Events must implement [`estoria.DomainEvent`](https://pkg.go.dev/github.com/go-estoria/estoria#DomainEvent):

```go
type UserNameChanged struct {
	NewName string
}

func (UserNameChanged) EventType() string { return "namechanged" }

func (UserNameChanged) New() estoria.DomainEvent[User] {
	return &UserNameChanged{NewName: "Unknown User"}
}

func (e UserNameChanged) ApplyTo(user User) User {
	user.Name = e.NewName
	return user
}
```

### Aggregates

Create an event store to store events:

```go
eventStore, _ := memory.NewEventStore()
```

Then, create an aggregate store using the event store, an aggregate type name, your entity factory function, and your event types. The type name becomes part of every aggregate's stream address, so it must remain stable for the lifetime of your data:

```go
aggregateStore, _ := aggregatestore.New(eventStore, "user", NewUser,
    aggregatestore.WithEventTypes(
        UserNameChanged{},
    ),
)
```

Now you can begin working with aggregates in your application:

```go
userID := uuid.Must(uuid.NewV4())

// create a new User aggregate
newUser := aggregateStore.New(userID)

// append an event
newUser.Append(UserNameChanged{NewName: "Juliette"})

// save the aggregate
_ = aggregateStore.Save(ctx, newUser, nil)

// load the aggregate
loadedUser, _ := aggregateStore.Load(ctx, userID, nil)

// access the aggregate's state
user := loadedUser.State()
```

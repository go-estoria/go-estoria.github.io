---
title: Getting Started
type: docs
prev: docs/introduction
next: docs/getting-started/defining-entities
weight: 120
---

Estoria requires Go >=1.25.

```shell
go get github.com/go-estoria/estoria
```

>The core Estoria components only depend on the Go standard library and `github.com/gofrs/uuid/v5` for UUID support. Vendor-specific components are imported separately as needed from [estoria-contrib](https://github.com/go-estoria/estoria-contrib).

## Quickstart

Run all of the code below in a [Go playground](https://goplay.tools/snippet/sB6aDoAoNS_f). See other full runnable examples in [estoria-examples](https://github.com/go-estoria/estoria-examples).

### Entities

Entities must implement [`estoria.Entity`](https://pkg.go.dev/github.com/go-estoria/estoria#Entity) and provide a factory function:

```go
type User struct {
	ID   uuid.UUID
	Name string
}

func (a User) EntityID() typeid.ID { return typeid.New("user", a.ID) }

func NewUser(id uuid.UUID) User {
	return User{ID: id, Name: "Unknown"}
}

```

### Events

Events must implement [`estoria.EntityEvent`](https://pkg.go.dev/github.com/go-estoria/estoria#EntityEvent):

```go
type UserNameChanged struct {
	NewName string
}

func (UserNameChanged) EventType() string { return "namechanged" }

func (UserNameChanged) New() estoria.EntityEvent[User] {
	return &UserNameChanged{NewName: "Unknown User"}
}

func (e UserNameChanged) ApplyTo(_ context.Context, user User) (User, error) {
	user.Name = e.NewName
	return user, nil
}
```

### Aggregates

Create an event store to store events:

```go
eventStore, _ := memory.NewEventStore()
```

Then, create an aggregate store using the event store, your entity factory function, and your event types:

```go
aggregateStore, _ := aggregatestore.New(eventStore, NewUser,
    aggregatestore.WithEventTypes(
        UserNameChanged{},
    ),
)
```

Now you can begin working with aggregates in your application:

```go
userID := uuid.Must(uuid.NewV4())

// create a new User aggregate
newUser := aggregateStore.New(userID, 0)

// append an event
_ = newUser.Append(UserNameChanged{NewName: "Juliette"})

// save the aggregate
_ = aggregateStore.Save(ctx, newUser, nil)

// load the aggregate
loadedUser, _ := aggregateStore.Load(ctx, userID, nil)

// access the aggregate root
user := loadedUser.Entity()
```

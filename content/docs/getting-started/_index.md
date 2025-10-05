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

## Quickstart

See full runnable examples in [estoria-examples](https://github.com/go-estoria/estoria-examples).

### Entities

Entities must implement [`estoria.Entity`](https://pkg.go.dev/github.com/go-estoria/estoria#Entity) and provide a factory func:

```go
type User struct {
	ID        uuid.UUID
	Name      string
}

func NewUser(id uuid.UUID) User {
    return User{ID: id, Name: "Unknown"}
}

func (a User) EntityID() typeid.ID { return typeid.New("user", a.ID) }
```

### Events

Events must implement [`estoria.EntityEvent`](https://pkg.go.dev/github.com/go-estoria/estoria#EntityEvent):

```go
type UserNameChanged struct {
	NewName string
}

func (UserNameChanged) EventType() string { return "namechanged" }

func (UserNameChanged) New() estoria.EntityEvent[User] {
	return UserNameChanged{NewName: "Unknown User"}
}

func (e UserNameChanged) ApplyTo(_ context.Context, user User) (User, error) {
	user.Name = e.NewName
	return user, nil
}
```

### Aggregates

Create an event store to store events. Then create an aggregate store using the event store, your entity factory func, and your event types:

```go
eventStore, _ := inmemoryeventstore.New()

aggregateStore, _ := aggregatestore.New(eventStore, NewUser,
    aggregatestore.WithEventTypes(
        UserNameChangedEvent{},
    ),
)
```

Now you can begin working with aggregates in your application:

```go
userID := uuid.Must(uuid.NewV4())

// create a new User aggregate
newUser := aggregatestore.New(NewUser(userID), 0)

// append an event
_ = newUser.Append(UserNameChangedEvent{NewName: "Juliette"})

// save the aggregate
_ = aggregateStore.Save(ctx, newUser, nil)

// load the aggregate
loadedUser, _ := aggregateStore.Load(ctx, userID, nil)

// access the aggregate root
user := loadedUser.Entity()
```

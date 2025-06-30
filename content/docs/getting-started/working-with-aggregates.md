---
title: Working with Aggregates
type: docs
prev: getting-started/creating-an-aggregate-store
next: getting-started/next-steps
weight: 125
---

Now that we have our aggregate store, we can begin working with aggregates in our application.

### Creating Aggregates and Appending Events

Create a new aggregate with `NewAggregate()`:

```go
userID := uuid.Must(uuid.NewV4())

newUser := aggregatestore.NewAggregate(NewUser(userID), 0)

fmt.Println("created new user:", newUser.Entity().Name)
```

We'll also append an event to change the User's name to "Juliette"

```go
if err := newUser.Append(
    UserNameChangedEvent{NewName: "Juliette"},
); err != nil {
    log.Fatalf("failed to append event to User: %v", err)
}
```

## Saving Aggregates

To save an aggregate, use `Save()`:

```go
if err := aggregateStore.Save(ctx, newUser, aggregatestore.SaveOptions{}); err != nil {
    log.Fatalf("failed to save aggregate: %v", err)
}

fmt.Println("saved user:", newUser.Entity().Name)
```

## Loading Aggregates

To load an aggregate, use `Load()`:

```go
loadedUser, err := aggregateStore.Load(ctx, userID, aggregatestore.LoadOptions{})
if err != nil {
    log.Fatalf("failed to load aggregate: %v", err)
}
```

The aggregate's root entity can be accessed using `.Entity()`:


```go
user := loadedUser.Entity()
fmt.Printf("loaded user", user.Name)
```

We can also see what version the aggregate is at (i.e. how many events have been applied to it):

```go
fmt.Printf("user %s is at version %d", user.Name, loadedUser.Version())
```

---
title: Working with Aggregates
type: docs
prev: getting-started/creating-an-aggregate-store
next: getting-started/next-steps
weight: 125
---

Now that we have our aggregate store, we can begin working with User aggregates in our application.

### Creating Aggregates and Appending Events

Create a new aggregate with `New()`:

```go
userID := uuid.Must(uuid.NewV4())

newUser := aggregateStore.New(userID)
```

We'll also append an event to change the User's name to "Juliette". Appending only queues the event on the aggregate; nothing is persisted or applied until the aggregate is saved:

```go
newUser.Append(UserNameChanged{NewName: "Juliette"})
```

## Saving Aggregates

To save an aggregate, use `Save()`:

```go
_ = aggregateStore.Save(ctx, newUser, nil)
```

Saving appends the queued events to the aggregate's event stream, then applies them to the in-memory state. If a save fails *after* its events were durably appended, the error carries the `aggregatestore.ErrEventsAppended` sentinel (check with `errors.Is`); recover by discarding the aggregate and reloading it.

## Loading Aggregates

To load an aggregate, use `Load()`:

```go
loadedUser, _ := aggregateStore.Load(ctx, userID, nil)
```

The aggregate's state can be accessed using `.State()`:

```go
user := loadedUser.State()
```

We can also see what version the aggregate is at (i.e. how many events have been applied to it):

```go
fmt.Printf("user %s is at version %d", user.Name, loadedUser.Version())
```

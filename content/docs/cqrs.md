---
title: CQRS
type: docs
prev: docs/telemetry/
weight: 820
---

CQRS stands for Command Query Responsibility Segregation. Simply put, it is an architectural pattern that involves separating the read concerns from the write concerns of a system. This separation allows for each side to be scaled and optimized independently. This may sometimes require dependent systems to be tolerant of eventual consistency.

CQRS is not event sourcing. However, event sourcing is often used in conjunction with CQRS.

Estoria does not provide any CQRS-specific components. However, it is fairly easy to use Estoria within a CQRS architecture. The following sections describe how to accomplish this.

## Basic Components

### Commands

A command is a request to perform an action. Commands exist on the write-side of a CQRS system and are sent to the system to change its state. Commands are typically sent via a command bus, which routes the command to the appropriate command handler.

With Estoria, your command handlers will typically load an aggregate, append one or more events, and then save the aggregate. Let's look at an example of an HTTP handler that handles a POST request to modify an account balance.

```go
type Account struct {
    ID      uuid.UUID
    Balance int
}

type ModifyBalanceCommand struct {
    AccountID uuid.UUID
    Amount    int
}

func (a *Application) HandleModifyBalance(r *http.Request, w *http.ResponseWriter) {
    cmd := &ModifyBalanceCommand{}
    json.NewDecoder(r.Body).Decode(cmd)

    // a.aggregates is an aggregate store for Accounts
    account, _ := a.aggregates.Load(ctx, cmd.AccountID)

    account.Append(&ModifyBalanceEvent{
        Amount: cmd.Amount,
    })

    a.aggregates.Save(ctx, account)

    w.WriteHeader(http.StatusNoContent)
}
```

### Queries

A query is a request to retrieve data. Queries exist on the read-side of a CQRS system and are used to retrieve data from the system.

With Estoria, you can leverage projections to build read models that can be queried.

#### Simple Projection

If your aggregates are comprised of relatively few events, you can often project a read model "on demand" directly from the events. Let's look at an example of an HTTP handler that handles a GET request and responds with a summary of an account.

```go
type AccountView struct {
    Balance int           `json:"balance"`
    LastUpdated time.Time `json:"last_updated"`
}

func (a *Application) GetAccountSummary(r *http.Request, w *http.ResponseWriter) {
    accountID := uuid.MustParse(r.URL.Query().Get("account_id"))

    // a.events is an event store containing Account events
    iter, _ := a.events.ReadStream(ctx, typeid.New("account", accountID)), eventstore.ReadStreamOptions{})

    projection, _ := projection.New(iter)

    account := &AccountView{}
    projection.Project(func(_ context.Context, event *eventstore.Event) {
        switch e := event.ID.Type {
        case "balancechanged":
            account.Balance += e.Amount
            account.LastUpdated = event.Timestamp
        }
    })

    json.NewEncoder(w).Encode(account)
}
```

However, this approach can become unwieldy as the number of events grows. In that case, you may want to build a dedicated read model persisted in a database.

#### Dedicated Read Model

If your aggregates are comprised of many events, or if you need to query the data in a different way, you may want to build a dedicated read model. Rather than projecting an in-memory view when a query is made, you instead project events to a database. This database is often entirely separate from the database that backs the event store and is optimized for queries (reads) rather than writes.

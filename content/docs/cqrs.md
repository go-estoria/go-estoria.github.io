---
title: CQRS
type: docs
prev: docs/telemetry/
next: docs/typeids/
weight: 820
---

Event sourcing is often used in conjunction with CQRS (Command Query Responsibility Segregation), an architectural pattern that separates the read concerns from the write concerns of a system. This separation allows for each side to be scaled and optimized independently. This may sometimes require dependent systems to be tolerant of eventual consistency.

Estoria does not provide any CQRS-specific components. However, it is fairly easy to use Estoria within a CQRS architecture. The following sections describe how to accomplish this.

## Basic Components

### Commands

A command is a request to change the state of the domain in some way. Thus, commands exist on the write-side of a CQRS system. Commands can be as simple as HTTP request handlers or pubsub message handlers, or involve command bus pipelines that deduplicate, validate, and authorize commands.

Regardless of how commands arrive in your application, with Estoria, your command handlers will typically load an aggregate, append one or more events, and then save the aggregate. Let's look at an example of an HTTP handler that handles a POST request to modify an account balance.

```go
type Application struct {
    accounts aggregatestore.Store[Account]
}

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

    account, _ := a.accounts.Load(ctx, cmd.AccountID)

    _ = account.Append(&ModifyBalanceEvent{
        Amount: cmd.Amount,
    })

    _ = a.accounts.Save(ctx, account, nil)

    w.WriteHeader(http.StatusNoContent)
}
```

### Queries

A query is a request to retrieve data. Queries exist on the read-side of a CQRS system and are used to obtain information from the system.

With Estoria, you can leverage [projections](../projections) to build read models that can be queried independently.

#### On-Demand Projections

If your aggregates are comprised of relatively few events, you can often project a read model "on demand" directly from the events. Let's look at an example of an HTTP handler that handles a GET request and responds with a summary of an account.

```go
type Application struct {
    events eventstore.Store
}

type AccountView struct {
    Balance int           `json:"balance"`
    LastUpdated time.Time `json:"last_updated"`
}

func (a *Application) HandleGetAccountSummary(r *http.Request, w *http.ResponseWriter) {
    accountID := uuid.MustParse(r.URL.Query().Get("account_id"))

    iter, _ := a.events.ReadStream(ctx, typeid.New("account", accountID)), eventstore.ReadStreamOptions{})

    projection, _ := projection.New(iter)

    account := &AccountView{}
    _ = projection.Project(func(_ context.Context, event *eventstore.Event) error {
        switch e := event.ID.Type {
        case "balancechanged":
            account.Balance += e.Amount
            account.LastUpdated = event.Timestamp
        }

        return nil
    })

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    _ = json.NewEncoder(w).Encode(account)
}
```

This approach can become unwieldy as the number of events grows, or as querying requirements become more complex. In that case, you should consider a dedicated read model persisted in a database.

#### Dedicated Read Models

If your aggregates are comprised of many events, or if you need more complex querying capabilities, you may want to build a dedicated read model. Rather than projecting an in-memory view when a query is made, you instead project events to a database whenever an aggregate changes. This database is often entirely separate from the database that backs the event store and is optimized for queries (reads) rather than writes.

---
title: TypeIDs
type: docs
prev: docs/cqrs/
weight: 830
---

Estoria uses a lightweight TypeID implementation to uniquely identify both aggregates and events. A TypeID is a tuple of `(string, uuid)`, where the string is a type name (e.g. "user", "order", "product") and the UUID is a unique identifier for a specific instance of that type. It is largely a convenience for transferring and presenting these two pieces of information as a single value.

```go
import "github.com/go-estoria/estoria/typeid"

id := typeid.NewV4("user")
// or
id := typeid.NewV7("user")
// or
id := typeid.New("user", uuid.Must(uuid.FromString("978e35ad-876f-43df-8e7d-cbcb6dd855f9")))

fmt.Println(id.Type)   // "user"
fmt.Println(id.UUID)   // 978e35ad-876f-43df-8e7d-cbcb6dd855f9
fmt.Println(id.String()) // "user_978e35ad-876f-43df-8e7d-cbcb6dd855f9"
fmt.Println(id.ShortString()) // "user_550e8400"
```

>Note that the Estoria `typeid` package is _not_ an implementation of [Jetify's TypeID specification](https://github.com/jetify-com/typeid/tree/main/spec). It is simply a struct that holds a type name and a UUID.
---
title: Component Library
---

## Event Store Implementations

| Backend | Driver | Outbox Support |
|---------|----------------|----------------|
| AWS DynamoDB | [AWS SDK for Go](https://github.com/aws/aws-sdk-go-v2) | Yes |
| Azure Cosmos DB | TBD | TBD |
| GCP Firestore | TBD | TBD |
| GCP Spanner | TBD | TBD |
| EventStoreDB | [EventStoreDB SDK](github.com/EventStore/EventStore-Client-Go) | No |
| MongoDB | [MongoDB](https://github.com/mongodb/mongo-go-driver) | Yes |
| SQL (agnostic) | `database/sql` (stdlib) | Yes |

## Snapshot Store Implementations

| Backend | Driver |
|---------|----------------|
| AWS DynamoDB | [AWS SDK for Go](https://github.com/aws/aws-sdk-go-v2) |
| Azure Cosmos DB | TBD |
| GCP Firestore | TBD |
| GCP Spanner | TBD |
| MongoDB | [MongoDB](https://github.com/mongodb/mongo-go-driver) |
| Redis | [Redis](https://github.com/go-redis/redis) |
| SQL (agnostic) | `database/sql` (stdlib) |
| Valkey | https://github.com/valkey-io/valkey-go |

## Aggregate Cache Implementations

| Backend | Type | Driver |
|---------|------|----------------|
| bigcache | In-Memory | https://github.com/allegro/bigcache |
| freecache | In-Memory | https://github.com/coocood/freecache |
| gocache | Multiple | https://github.com/eko/gocache |
| Redis | Distributed | https://github.com/go-redis/redis |
| Valkey | Distributed | https://github.com/valkey-io/valkey-go |

# Шаг 3 — API Design

← [FRAMEWORK](../FRAMEWORK.md)

## Эндпоинты

| Метод | UC | Sync/Async | Заметка |
|-------|-----|------------|---------|
| `POST /posts` | UC1 | sync | idempotency key |
| `POST /media` | UC1 | sync | presigned upload |
| `GET /feed` | UC2 | sync | offset / cursor · push/pull |
| `POST /follow` | UC4 | sync | — |

## Trade-offs

| Тема | Файл |
|------|------|
| REST / gRPC / GraphQL | [rest-grpc-graphql](../trade-offs/api/rest-grpc-graphql.md) |
| Sync / async | [sync-async](../trade-offs/api/sync-async-messaging.md) |
| RPC / queue | [rpc-vs-queue](../trade-offs/api/rpc-vs-queue.md) |
| Push / pull | [push-pull](../trade-offs/api/push-vs-pull-delivery.md) |
| WS / SSE | [realtime](../trade-offs/api/realtime-transport.md) |
| Idempotency | [idempotency](../trade-offs/api/write-api-idempotency.md) |
| Versioning | [versioning](../trade-offs/api/api-versioning-evolution.md) |
| Pagination | [pagination](../trade-offs/data/pagination-cursor-offset.md) |

---

← [02 — NFR](02-non-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · [04 — Data](04-data-model.md) →

Пример: [instagram-feed.md](../examples/instagram-feed.md)

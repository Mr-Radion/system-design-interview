# Шаг 5 — High-Level Design

← [FRAMEWORK.md](../FRAMEWORK.md)

## Схема

```
Client → DNS → L7 Gateway (JWT, rate limit)
              → Feed Service → Redis (feed cache)
              → Post Service → PostgreSQL
              → Media Service → S3 → CDN
              → Social Service → PostgreSQL (follows)
         Kafka ← publish post → Feed Worker (fan-out)
```

## Data flow: опубликовать пост

| # | Что происходит |
|---|----------------|
| 1 | Client `POST /posts` + idempotency key |
| 2 | Post Service пишет в PostgreSQL |
| 3 | Событие в Kafka |
| 4 | Feed Worker fan-out в Redis Lists подписчиков |
| 5 | Client 201 — пост создан (в ленте подписчиков через 1–3s) |

## Data flow: открыть ленту

| # | Что происходит |
|---|----------------|
| 1 | `GET /feed` → Feed Service |
| 2 | Read post_ids из Redis |
| 3 | Hydrate метаданные из PostgreSQL / cache |
| 4 | URL фото — CDN |

**При сбое Feed Service:** circuit breaker → stale feed из Redis (см. [шаг 2](02-non-functional-requirements.md)).

---

← Назад: [04 — Data](04-data-model.md) · [FRAMEWORK](../FRAMEWORK.md)

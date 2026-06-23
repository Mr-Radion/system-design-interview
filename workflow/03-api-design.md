# Шаг 3 — API Design

← [FRAMEWORK.md](../FRAMEWORK.md)

**Пример:** лента постов · REST для клиента

| Метод | Назначение |
|-------|------------|
| `POST /posts` | Создать пост (текст + `media_id`) |
| `POST /media/upload` | Presigned URL для фото |
| `GET /feed?cursor=` | Лента, 10 постов, cursor pagination |
| `POST /posts/{id}/like` | Лайк (`X-Idempotency-Key`) |
| `POST /posts/{id}/comments` | Комментарий |
| `POST /users/{id}/follow` | Подписка / отписка |

**Заголовки write:** `Authorization: Bearer`, `X-Idempotency-Key`

**Sync vs async:** клиент ждёт 201 на пост; fan-out в ленты — **async** (Kafka), из [шага 2](02-non-functional-requirements.md).

---

← Назад: [02 — NFR](02-non-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · Дальше: [04 — Data](04-data-model.md) →

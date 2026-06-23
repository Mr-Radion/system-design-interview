# Шаг 3 — API Design

← [FRAMEWORK](../FRAMEWORK.md)

## Что фиксируем

| Метод | UC | Sync/Async | Заметка |
|-------|-----|------------|---------|
| `POST /…` | UC1 | sync | idempotency key |
| `GET /…` | UC2 | sync | pagination |
| … | … | async | очередь / webhook |

**Заголовки write:** `Authorization`, `X-Idempotency-Key` (если нужно).

**Протокол:** REST для клиента · gRPC между сервисами · WebSocket если real-time.

---

← Назад: [02 — NFR](02-non-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · Дальше: [04 — Data](04-data-model.md) →

Пример: [instagram-feed.md](../examples/instagram-feed.md)

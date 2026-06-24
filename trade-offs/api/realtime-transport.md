---
layer: paradigm
steps: [3, 5]
related:
  - architecture/stateless-stateful
  - api/push-vs-pull-delivery
---

# Real-time транспорт (транспорт реального времени)

> **Главное:** Real-time transport — выбор транспортного протокола. Вход — UX latency + connection model.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Двусторонняя связь нужна? | Да | WebSocket / WebRTC |
| Ограничен браузерный SSE? | Да | SSE |
| Нужен high-throughput pub/sub? | Да | MQTT / Kafka bridge |

## Цепочка решений

шаг 2 NFR → HLD §3.1 / Deep Dive §4.1 → Deep Dive §4.x → Deep Dive §4.x (tech)

## WebSockets (Full-Duplex)

- ➕ **Плюсы:** минимальный overhead после handshake, двусторонний real-time, low latency.
- ➖ **Минусы / Цена:** stateful connections; sticky sessions; horizontal scale → Pub/Sub (Publish/Subscribe, публикация/подписка) (Redis); LB (Load Balancer, балансировщик нагрузки) complexity.
- 📍 **Где применять:** chat, multiplayer games, collaborative editing, live trading.

## SSE (Server-Sent Events)

- ➕ **Плюсы:** server→client push over HTTP (HyperText Transfer Protocol); auto-reconnect; проще WebSocket; работает через HTTP/2.
- ➖ **Минусы / Цена:** только однонаправленный (server→client); лимит connections в браузере.
- 📍 **Где применять:** live scores, stock tickers, notification stream (read-only updates).

## Long Polling (долгий опрос)

- ➕ **Плюсы:** stateless-ish; работает везде; проще за corporate firewalls.
- ➖ **Минусы / Цена:** overhead на reopen connections; latency выше WebSocket; server resources на hanging requests.
- 📍 **Где применять:** legacy support, simple notification fallback.

## Short Polling

- ➕ **Плюсы:** тривиальная реализация.
- ➖ **Минусы / Цена:** 100K users × poll/10s = 10K RPS (Requests Per Second, запросов в секунду) waste (Medium dashboard example).
- 📍 **Где применять:** dev/MVP only, very low traffic.

## Матрица

| | Duplex | Stateful | Scale difficulty |
|--|--------|----------|------------------|
| WebSocket | ✅ | ✅ | Hard |
| SSE (Server-Sent Events, события от сервера) | ❌ (S→C) | Partial | Medium |
| Long Polling | ❌ | Partial | Medium |
| Short Polling | ❌ | ❌ | Easy |

**Medium:** Long polling = repeatedly asking. WebSocket = sitting on a call.

## Realtime Pub/Sub (Publish/Subscribe, публикация/подписка) vs Durable Stream (для multi-server chat)

| | Redis Pub/Sub (Publish/Subscribe, публикация/подписка) | Kafka / durable stream |
|--|---------------|------------------------|
| Persistence | ❌ fire-and-forget | ✅ retention, replay |
| Latency | Ultra-low | Higher |
| Multi-server fan-out | ✅ идеально для chat relay | Overkill |
| Message loss if no subscriber | ✅ possible | ❌ retained |

- ➕ **Redis Pub/Sub (Publish/Subscribe, публикация/подписка):** Client-1 → Server-1 → Redis channel → Server-2 → Client-3; push model, no polling.
- ➖ **Redis Pub/Sub (Publish/Subscribe, публикация/подписка):** no persistence — offline subscriber misses messages; not for billing/events.

---

## Примечания (Habr, части 3 и 5)

### Multi-server WebSocket + Redis Pub/Sub (Publish/Subscribe, публикация/подписка)

Проблема: Client-1 на Server-1, Client-3 на Server-2 — прямой push невозможен.

```
Client-1 → WS → Server-1 → PUBLISH redis:chat:room123 → Server-2 → WS → Client-3
```

**Redis Pub/Sub (Publish/Subscribe, публикация/подписка):** push-модель, сообщение не сохраняется — доставлено подписчикам online сейчас. Для offline history нужен отдельный store (DB).

### Когда WebSocket + Pub/Sub (Publish/Subscribe, публикация/подписка) vs только WebSocket

- 1 server → Pub/Sub (Publish/Subscribe, публикация/подписка) не нужен.
- N servers behind LB (Load Balancer, балансировщик нагрузки) → **обязателен** message bus (Redis Pub/Sub (Publish/Subscribe, публикация/подписка), NATS) или sticky sessions (хуже для scale).

**Источники:** [часть 3](https://habr.com/ru/articles/885054/), [часть 5](https://habr.com/ru/articles/900396/)


---

## Сокращения в этом файле

| Аббревиатура | Расшифровка |
|--------------|-------------|
| FR | Functional Requirements — функциональные требования |
| NFR | Non-Functional Requirements — нефункциональные требования |
| HLD | High-Level Design — высокоуровневый дизайн |
| RPS | Requests Per Second — запросов в секунду |
| CAP | Consistency, Availability, Partition tolerance |
| LB | Load Balancer — балансировщик нагрузки |

Полный индекс: [README.md](../../FRAMEWORK.md#trade-offs).

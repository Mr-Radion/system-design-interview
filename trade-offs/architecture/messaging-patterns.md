---
layer: pattern
steps: [5]
related:
  - technologies/message-brokers
  - api/sync-async-messaging
---

# Messaging Patterns (паттерны обмена сообщениями)

> **Главное:** Messaging patterns — выбор pub/sub, event sourcing, outbox. Вход — coupling and delivery guarantees.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need durable ordered stream? | Да | Kafka |
| Need flexible routing? | Да | RabbitMQ / broker |

## Цепочка решений

Шаг 2 NFR → шаг 5 messaging pattern → шаг 2 Infra tech

## Queue (Point-to-Point)

**Выбор:** one consumer per message (competing consumers).

- ➕ **Плюсы:** load distribution; task queue semantics; simple mental model.
- ➖ **Минусы / Цена:** no broadcast; ordering per queue not global.
- 📍 **Где применять:** order processing, email jobs, image resize workers.

## Pub/Sub (Publish/Subscribe, публикация/подписка) (Topic)

**Выбор:** N subscribers each get copy of message.

- ➕ **Плюсы:** fan-out; decoupled event notification; multiple services react.
- ➖ **Минусы / Цена:** duplicate processing; harder ordering guarantees.
- 📍 **Где применять:** user signed up → email + analytics + CRM.

## Stream (Log-based, Kafka-style)

**Выбор:** durable ordered log, consumers track offset.

- ➕ **Плюсы:** replay; high throughput; event sourcing backbone; retention.
- ➖ **Минусы / Цена:** operational complexity; partition tuning; not for simple task queue.
- 📍 **Где применять:** event sourcing, activity feed pipeline, CDC (Change Data Capture, захват изменений).

## Delivery Semantics

| Semantic | Guarantee | Cost |
|----------|-----------|------|
| At-most-once | May lose | Cheapest |
| At-least-once | May duplicate | + idempotency |
| Exactly-once | No dup, no loss | Hardest (Kafka transactions) |

## Event Notification vs Event-Carried State Transfer (ECST (Event-Carried State Transfer, состояние в событии)

| Pattern | Payload | Coupling |
|---------|---------|----------|
| Notification | Event ID only | Consumer fetches state |
| ECST (Event-Carried State Transfer, состояние в событии) | Full state in event | Self-contained, larger messages |

## Event Sourcing

**Выбор:** хранить state как log of events, не current row.

- ➕ **Плюсы:** full audit trail; replay; temporal queries («state at T»).
- ➖ **Минусы / Цена:** complex reads (project state); event schema evolution; storage growth.
- 📍 **Где применять:** banking ledger, compliance, collaborative docs.

## CQRS (Command Query Responsibility Segregation)

**Выбор:** отдельные модели для write (commands) и read (queries).

- ➕ **Плюсы:** optimize read/write independently; scale read side separately.
- ➖ **Минусы / Цена:** eventual consistency between models; more moving parts.
- 📍 **С Event Sourcing:** natural pair. **Без ES:** CQRS (Command Query Responsibility Segregation, разделение команд и запросов)-lite (read replicas + denormalized views).

## Queue vs Stream — когда что (Habr)

| | Message Queue | Message Stream |
|--|---------------|----------------|
| Consumers | One consumer deletes message | Many consumer groups, same message |
| Delete | Consumer removes after process | Messages retained (TTL (Time To Live, время жизни)/manual) |
| Use | Task queue (transcode video) | «Write once, read many» (transcode + subtitles) |

**Проблема двух queues:** producer crash между queue-1 и queue-2 → inconsistency. **Stream** решает: оба consumer groups читают один log.

**Шаг 5:** pattern. см. [Infra-таблицу шага 2](../../workflow/02-non-functional-requirements.md).

---

## Примечания (Habr, части 4–5)

### EDA (Event-Driven Architecture, событийная архитектура) — e-commerce example

Sync chain: Order → Payment → Inventory + Notification блокирует user на success page.

**EDA (Event-Driven Architecture, событийная архитектура) fix:** Order service публикует `OrderCompleted {order_id}` → Inventory и Notification consume async. User redirect не ждёт inventory/email.

**Benefits (из статьи):** decoupling (inventory down ≠ order down), resilience, independent scale.

### Event Notification vs ECST (Event-Carried State Transfer, состояние в событии) — нагрузка

| Pattern | Payload | DB load | Bandwidth |
|---------|---------|---------|-----------|
| Notification `{order_id: 42}` | Minimal | Consumer SELECT order | Low |
| ECST (Event-Carried State Transfer, состояние в событии) `{order_id, items, user, total...}` | Full | No extra fetch | High |

**Trade-off ECST (Event-Carried State Transfer, состояние в событии):** lower latency (no extra network hop to DB) vs larger messages + storage cost.

Celery-пример из перевода: task с id vs task с full dict — тот же trade-off.

### 4 паттерна EDA (полный список Habr)

1. Event Notification (id only)
2. Event-Carried State Transfer (full state)
3. Event Sourcing
4. Event Sourcing + CQRS (Command Query Responsibility Segregation, разделение команд и запросов)

**Источники:** [часть 4](https://habr.com/ru/articles/893548/), [часть 5](https://habr.com/ru/articles/900396/)


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

---
layer: paradigm
steps: [3]
related:
  - architecture/cap-pacelc-distributed
---

# Idempotency Write-API (идемпотентность операций записи)

> **Главное:** Idempotency for write APIs — защита от дупликатов. Вход — at-least-once delivery, retries.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Возможны дупликаты? | Да | Idempotency keys |
| Нужна транзакционная согласованность? | Да | Outbox + TX |

## Цепочка решений

шаг 2 NFR → HLD §3.1 + Deep Dive §4.3 → Deep Dive §4.3 outbox/idempotency → Deep Dive §4.x (tech)

## Зачем

Double-click «Order», network retry, mobile reconnect → без idempotency = duplicate orders/charges.

## Idempotency Key (рекомендуется)

**Выбор:** клиент шлёт уникальный `Idempotency-Key` / `request_id` с каждым write.

- ➕ **Плюсы:** стандарт (Stripe, PayPal); client-controlled; safe retries.
- ➖ **Минусы / Цена:** storage keys + results (TTL (Time To Live, время жизни) 24–72h); lookup latency ~1ms.
- 📍 **Где применять:** payments, order creation, booking.

## Dedup Table

**Выбор:** server-side hash входящего payload.

- ➕ **Плюсы:** не зависит от client discipline.
- ➖ **Минусы / Цена:** collision risk; не различает intentional re-submit vs retry.
- 📍 **Где применять:** webhook receivers, event ingestion.

## Operation Log (Event Sourcing lite)

- ➕ **Плюсы:** full audit trail; replay capability.
- ➖ **Минусы / Цена:** storage growth; complexity.
- 📍 **Где применять:** banking, compliance-heavy systems.


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

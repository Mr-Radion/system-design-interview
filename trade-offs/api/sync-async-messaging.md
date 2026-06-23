---
layer: paradigm
steps: [3]
related:
  - architecture/messaging-patterns
  - api/rpc-vs-queue
---

# Sync vs Async (синхронное vs асинхронное взаимодействие)

> **Главное:** Sync vs Async — выбор между немедленным ответом и decoupled delivery. Вход — шаг 2 NFR latency/consistency.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Критично ответить синхронно? | Да | Sync API |
| Можно ли ретраями/идемпотентностью? | Да | Async messaging |

## Цепочка решений

Шаг 2 NFR → шаг 3 API (sync/async) → шаг 5 паттерн → шаг 2 Infra технологи

## Synchronous (Request-Response)

**Выбор:** прямой вызов HTTP (HyperText Transfer Protocol)/gRPC, клиент ждёт ответ.

- ➕ **Плюсы:** простота отладки, линейный код, мгновенный результат (success/error), linearizability UX (User Experience, пользовательский опыт).
- ➖ **Минусы / Цена:** tight coupling; downstream down → cascade failure; latency суммируется по цепочке.
- 📍 **Где применять:** payment confirm, auth check, read user profile.

## Asynchronous (Event-Driven / Message Broker)

**Выбор:** отправка события в шину, обработка позже.

- ➕ **Плюсы:** loose coupling, backpressure, буфер при пиках, приёмник down → broker хранит message.
- ➖ **Минусы / Цена:** eventual consistency, сложный debug (нужен tracing), пользователь не видит результат сразу.
- 📍 **Где применять:** email notifications, video transcoding, order fulfillment pipeline.


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

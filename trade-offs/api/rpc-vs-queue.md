---
layer: paradigm
steps: [3]
related:
  - api/sync-async-messaging
  - architecture/messaging-patterns
---

# RPC (Remote Procedure Call, удалённый вызов процедуры) vs Queue (удалённый вызов vs очередь сообщений)

> **Главное:** RPC vs Queue — trade-off между синхронностью и долговечностью доставки. Вход — шаг 2 NFR.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужен транзакционный ответ? | Да | RPC/sync |
| Нужна гарантия доставки и decoupling? | Да | Queue/async |

## Цепочка решений

шаг 2 NFR → HLD §3.1 API → Deep Dive §4.3 outbox/queue → Deep Dive §4.x (tech)

## RPC (Remote Procedure Call)

**Выбор:** синхронный вызов, ответ в рамках одного request.

- ➕ **Плюсы:** deterministic result, immediate UX (User Experience, пользовательский опыт) feedback, простая error handling для пользователя.
- ➖ **Минусы / Цена:** caller blocked; timeout management; cascade failures.
- 📍 **Где применять:** payment gateway, fraud check, real-time price quote.

## Message Queue

**Выбор:** fire-and-forget, consumer обрабатывает асинхронно.

- ➕ **Плюсы:** decoupling, load leveling, retry без блокировки caller.
- ➖ **Минусы / Цена:** нет мгновенного ответа; idempotency required; harder tracing.
- 📍 **Где применять:** send email, generate PDF report, resize image, index document.


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

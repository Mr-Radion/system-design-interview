---
layer: paradigm
steps: [3]
related:
  - api/realtime-transport
  - architecture/observability-architecture
---

# Push vs Pull (сервер шлёт vs клиент опрашивает)

> **Главное:** Push vs Pull — модель доставки уведомлений и данных. Вход — client connectivity & scale.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Клиент всегда online? | Да | Push |
| Клиент sporadic / polling cheap? | Да | Pull |

## Цепочка решений

Шаг 2 NFR → шаг 3 API delivery → шаг 5 pattern → шаг 2 Infra tech

## Pull (клиент опрашивает)

- ➕ **Плюсы:** простота; клиент контролирует частоту; stateless server.
- ➖ **Минусы / Цена:** wasted requests; 100K users × 10s poll = 10K RPS (Requests Per Second, запросов в секунду); высокий infra cost.
- 📍 **Где применять:** low-frequency updates, admin dashboards, batch status check.

## Push (сервер отправляет)

- ➕ **Плюсы:** efficient; updates only when needed; lower bandwidth at scale.
- ➖ **Минусы / Цена:** connection management; stateful infra; reconnect logic.
- 📍 **Где применять:** delivery tracking, live dashboard, notifications at scale.

**Medium example:** delivery status dashboard — polling in dev OK; prod → WebSocket push saved thousands in infra.

**Не путать с:** Push/Pull metrics (Prometheus) → [observability-architecture](../architecture/observability-architecture.md) — другой контекст, тот же принцип.


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

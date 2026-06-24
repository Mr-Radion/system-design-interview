---
layer: pattern
steps: [5]
related:
  - constraints/vertical-vs-horizontal-scaling
  - api/realtime-transport
---

# Stateless vs Stateful (без состояния vs с состоянием)

> **Главное:** Stateless vs Stateful — где хранить state. Вход — session model и scaling needs.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Можно ли вынести state в DB/Redis? | Да | Stateless app |
| State per node required? | Да | Stateful services |

## Цепочка решений

шаг 2 NFR → HLD §3.3 / Deep Dive §4.4 → Deep Dive §4.x (tech)

## Stateless

**Выбор:** сервер не хранит user context in memory; всё из DB/Cache per request.

- ➕ **Плюсы:** perfect horizontal scale; any server handles any request; kill & replace pods freely.
- ➖ **Минусы / Цена:** network round-trip to DB/Redis every request; session data externalized.
- 📍 **Где применять:** REST APIs, most microservices, 12-factor apps.

## Stateful

**Выбор:** сервер держит session/game state in local RAM (Random Access Memory, оперативная память).

- ➕ **Плюсы:** zero network for state access; ultra-low latency; natural for games.
- ➖ **Минусы / Цена:** sticky sessions required; server death = lost sessions; hard to scale; uneven load.
- 📍 **Где применять:** multiplayer game servers, WebSocket rooms, streaming aggregations.

**Medium example:** multiplayer game + REST = failed; moved to persistent sockets + stateful sessions.

## Sticky Sessions vs Stateless Routing

| | Sticky | Stateless |
|--|--------|-----------|
| LB (Load Balancer, балансировщик нагрузки) config | Cookie/IP hash | Round robin |
| Scale | Hard | Easy |
| Failover | Session lost | Transparent |

**Правило:** default stateless; stateful only when latency requirement demands it.


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

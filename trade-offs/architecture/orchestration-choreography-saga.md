---
layer: pattern
steps: [5]
related:
  - api/sync-async-messaging
---

# Orchestration vs Choreography / Saga (оркестрация vs хореография / сага)

> **Главное:** Orchestration vs Choreography / Saga — управление распределёнными трансакциями.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need central coordinator? | Да | Orchestration |
| Prefer decentralized events? | Да | Choreography / Saga |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.3 coordination → Deep Dive §4.x (tech)

## Orchestration

**Выбор:** central coordinator (дирижёр) вызывает сервисы по шагам.

- ➕ **Плюсы:** process logic in one place; easy to read state; simple monitoring of workflow.
- ➖ **Минусы / Цена:** orchestrator = SPOF (Single Point Of Failure, единая точка отказа) + bottleneck; tight coupling to center.
- 📍 **Где применять:** complex multi-step flows with clear sequence (order saga with Temporal/Camunda).

## Choreography

**Выбор:** no central coordinator; services react to events independently.

- ➕ **Плюсы:** loose coupling; no SPOF (Single Point Of Failure, единая точка отказа); natural scale.
- ➖ **Минусы / Цена:** process «размазан» по сервисам; hard to debug; compensation chains complex.
- 📍 **Где применять:** simple event chains, mature event-driven org.

## Saga: 2PC (Two-Phase Commit, двухфазный коммит) vs Choreographed vs Orchestrated

| Approach | Consistency | Complexity |
|----------|-------------|------------|
| 2PC (Two-Phase Commit) | Strong | Blocking, doesn't scale |
| Orchestrated Saga | Eventual | Compensating transactions |
| Choreographed Saga | Eventual | Implicit, hard to trace |

## Compensating Transaction Example

```
CreateOrder → ReserveInventory → ChargePayment
  ↓ fail at Charge
  → ReleaseInventory (compensate)
  → CancelOrder (compensate)
```

**Правило:** avoid 2PC (Two-Phase Commit, двухфазный коммит) in microservices; use Saga + idempotency.


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

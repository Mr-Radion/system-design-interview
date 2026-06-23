# Шаг 5 — High-Level Design

← [FRAMEWORK](../FRAMEWORK.md)

## Trade-offs (перед схемой)

| Тема | Файл |
|------|------|
| Monolith / micro | [monolith-micro](../trade-offs/architecture/monolith-microservices.md) |
| Stateless / stateful | [stateless](../trade-offs/architecture/stateless-stateful.md) |
| L4 / L7 | [lb](../trade-offs/architecture/load-balancing-l4-l7.md) |
| Messaging | [messaging](../trade-offs/architecture/messaging-patterns.md) |
| Saga / outbox | [saga-outbox](../trade-offs/architecture/saga-vs-outbox.md) |
| Orchestration / choreography | [orchestration](../trade-offs/architecture/orchestration-choreography-saga.md) |
| Concurrency / parallelism | [concurrency](../trade-offs/architecture/concurrency-vs-parallelism.md) |
| Coordination | [coordination](../trade-offs/architecture/distributed-coordination.md) |
| ETL | [etl](../trade-offs/architecture/etl-pipeline-pattern.md) |

## Схема

```
Client → Gateway → Services → DB / Cache / Queue
```

## Data flow (1 UC)

| # | Компонент | Действие |
|---|-----------|----------|
| 1 | … | … |
| 2 | … | … |

**При сбое:** деградация из trade-offs шага 2 (Reliability).

---

← [04 — Data](04-data-model.md) · [FRAMEWORK](../FRAMEWORK.md)

Пример: [instagram-feed.md](../examples/instagram-feed.md)

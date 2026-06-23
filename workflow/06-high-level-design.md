# Шаг 6 — High-Level Design

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

Шаблон: **§5 arch choices → диаграмма** (см. [пример](../examples/instagram-feed.md#6-hld)).

### Общая

```mermaid
flowchart LR
    Client --> Gateway
    Gateway --> Services
    Services --> DB
    Services --> Cache
    Services --> Queue
```

### Data layer

```mermaid
flowchart LR
    Writers --> Master[("Master")]
    Master -->|HA replicate| Replicas[("Replicas")]
    Replicas --> Readers[Readers]
    Services --> Router[Shard Router]
    Router --> Shards[("Shard 1..N")]
    Services --> Cache[("Cache")]
    Cache -->|miss| Replicas
```

## Data flow (1 UC)

| # | Компонент | Действие |
|---|-----------|----------|
| 1 | … | … |
| 2 | … | … |

**При сбое:** деградация из §5 failure modes.

---

← [05 — Arch characteristics](05-architectural-characteristics.md) · [FRAMEWORK](../FRAMEWORK.md) · [07 — Technology](07-technology-choices.md) →

Примеры: [instagram-feed.md](../examples/instagram-feed.md) · [paypal-payments.md](../examples/paypal-payments.md) · [vk-social.md](../examples/vk-social.md)

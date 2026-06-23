# Шаг 2 — NFR + trade-offs

← [FRAMEWORK](../FRAMEWORK.md)

## Что фиксируем

Цифры с собеса + NFR по подсекциям (допущения → capacity → CAP → latency → throughput → availability → observability). **Без имён технологий** — только требования и SLO.

**Infra и деревья решений** — после шага 5 (HLD), см. [example §6](../examples/instagram-feed.md#6-technology-choices).

## Расчёты

```
Read RPS   = users × reads/day ÷ 86_400
Write RPS  = users × writes/day ÷ 86_400
Read Mbps  = read_RPS × постов_в_ответе × размер_поста
Write Mbps = write_RPS × размер_поста
Storage/год = write_Mbps × 86_400 × 365
```

## Latency — порядок величин (back-of-envelope)

| Store / hop | ~latency |
|-------------|----------|
| RAM / L1 cache | µs |
| SSD | ~1 ms |
| HDD | ~10 ms |
| Same DC network | ~0.5 ms |
| Cross-region | ~50–150 ms |

## Чеклист + trade-offs

| Блок | Темы | Trade-offs |
|------|------|------------|
| **Performance** | Latency SLA · RPS · Capacity · CDN · cost | [latency](../trade-offs/constraints/latency-vs-throughput.md) · [cost](../trade-offs/constraints/performance-vs-cost.md) · [CDN](../trade-offs/architecture/cdn-object-storage-pattern.md) |
| Scalability | vertical/horizontal · scalability vs perf · autoscaling · cache | [scaling](../trade-offs/constraints/vertical-vs-horizontal-scaling.md) · [scal vs perf](../trade-offs/constraints/scalability-vs-performance.md) · [autoscale](../trade-offs/constraints/autoscaling-vs-fixed-capacity.md) · [cache](../trade-offs/architecture/caching-patterns.md) |
| Consistency | CAP · PACELC · strong/eventual | [CAP](../trade-offs/architecture/cap-pacelc-distributed.md) · [consistency](../trade-offs/constraints/consistency-as-nfr.md) |
| Reliability | SLA · RPO/RTO · DR · circuit breaker · backpressure | [SLO/RPO](../trade-offs/constraints/availability-slo-rpo-rto.md) · [DR](../trade-offs/architecture/disaster-recovery-pattern.md) · [resilience](../trade-offs/architecture/resilience-backpressure.md) |
| Observability | metrics · logs · traces | [observability](../trade-offs/architecture/observability-architecture.md) · [monitoring](../trade-offs/technologies/monitoring-tools.md) |
| Processing | batch vs stream (Cron) | [batch/stream](../trade-offs/architecture/batch-vs-stream.md) |
| Security | auth · rate limit · signed URL | [gateway](../trade-offs/technologies/api-gateways.md) |
| **Infra** | DB · cache · broker · S3 · gateway · LB · monitoring | [meta](../trade-offs/technologies/technology-selection-meta.md) · [DB](../trade-offs/technologies/databases.md) · [cache](../trade-offs/technologies/caches.md) · [broker](../trade-offs/technologies/message-brokers.md) · [S3](../trade-offs/technologies/object-storage.md) · [gateway](../trade-offs/technologies/api-gateways.md) · [LB](../trade-offs/technologies/load-balancers-proxies.md) |

## Формат одной темы

> **Трейдоф** → **A** / **B** → **✅ Выбор**

## Infra — после HLD (шаг 5 → example §6)

Дерево решений на каждый класс (broker, cache, DB…) + итоговая таблица с продуктами и sizing. Шаблон — [instagram-feed §6](../examples/instagram-feed.md#6-technology-choices).

---

← [01 — FR](01-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · [03 — API](03-api-design.md) →

Пример: [instagram-feed.md](../examples/instagram-feed.md) · [vk-social.md](../examples/vk-social.md)

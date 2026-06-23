# Шаг 5 — Архитектурные характеристики

← [FRAMEWORK](../FRAMEWORK.md)

## Что фиксируем

**Trade-offs и паттерны** — без имён продуктов. Цифры из §2 → здесь **решения**. Infra names → §7.

Источник категорий: Highload part 2 (Operational / Structural / Cross-cutting).

## Шаблон §5

| Категория | Характеристика | ✅ Выбор | Trade-off | Почему (FR) |
|-----------|----------------|----------|-----------|-------------|
| **Operational** | Availability | async repl + failover | [replication](../trade-offs/data/replication-sync-async.md) | HA, **не read scale** |
| | Continuity | rolling deploy | [deployment](../trade-offs/architecture/deployment-release-strategies.md) | zero-downtime |
| | DR | RPO/RTO из §2.3 | [DR](../trade-offs/architecture/disaster-recovery-pattern.md) | backup, failover |
| **Structural** | Scalability | CDN + cache + shard | [cache](../trade-offs/architecture/caching-patterns.md) · [sharding](../trade-offs/data/sharding-partitioning.md) | read vs write |
| | Consistency (CAP) | strong / eventual по участку | [CAP](../trade-offs/architecture/cap-pacelc-distributed.md) | FR-… |
| **Cross-cutting** | Caching | cache-aside / write-behind | [caching-patterns](../trade-offs/architecture/caching-patterns.md) | read throughput |
| | Observability | metrics из §2.5 | [observability](../trade-offs/architecture/observability-architecture.md) | SLO |

### Failure modes

| Сбой | Поведение | FR |
|------|-----------|-----|
| … | … | FR-… |

### Traceability (FR → §5 → §7)

| FR | Arch choice §5 | Tech §7 |
|----|----------------|---------|
| FR-… | … | … |

## Правило репликации

**Replication = отказоустойчивость (HA, DR, RPO/RTO), не масштабирование.**

| Цель | Механизм |
|------|----------|
| Read throughput / latency | CDN, cache-aside, stateless app + LB |
| Write throughput / storage | Sharding, partition |
| HA / failover | Async/sync replication, promoted replica |

→ [master-slave](../trade-offs/data/master-slave-multi-master.md) · [vertical-vs-horizontal](../trade-offs/constraints/vertical-vs-horizontal-scaling.md)

---

← [04 — Data](04-data-model.md) · [FRAMEWORK](../FRAMEWORK.md) · [06 — HLD](06-high-level-design.md) →

Примеры: [instagram §5](../examples/instagram-feed.md#5-architectural-characteristics) · [paypal §5](../examples/paypal-payments.md#5-architectural-characteristics) · [vk-social §5](../examples/vk-social.md#5-architectural-characteristics)

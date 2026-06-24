---
layer: pattern
steps: [4]
related:
  - constraints/consistency-as-nfr
  - data/master-slave-multi-master
---

# Sync vs Async Replication (синхронная vs асинхронная репликация)

> **Главное:** Replication — **отказоустойчивость (HA, DR)**, не масштабирование. Sync vs async — по RPO/RTO (шаг 2 NFR → Deep Dive §4.x).

## Replication ≠ scaling

| Цель | Механизм |
|------|----------|
| HA, failover, RPO/RTO | **Replication** (sync / async / semi-sync) |
| Read throughput, latency | **CDN, cache-aside**, stateless app + LB |
| Write throughput, storage size | **Sharding**, partition |

Read с replica — **побочный offload** с replication lag, не primary reason для repl.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужен RPO≈0? | Да | Sync replication |
| Можно ли иметь stale reads? | Да | Async replication |

## Цепочка решений

шаг 2 NFR (RPO/RTO) → Deep Dive §4.4 + §4.2 → Deep Dive §4.x (tech)

## Synchronous Replication

**Выбор:** master ждёт ACK от replica(s) перед commit.

- ➕ **Плюсы:** zero data loss on master fail; replicas always up-to-date; strong consistency reads from replica.
- ➖ **Минусы / Цена:** write latency ↑ (network RTT (Round Trip Time, время кругового пути); one slow replica blocks all writes; availability ↓ on partition.
- 📍 **Где применять:** financial ledger, inventory with strict consistency.

## Asynchronous Replication

**Выбор:** master commits locally, replicas catch up in background.

- ➕ **Плюсы:** minimal write latency; master availability; geo-distributed replicas cheap.
- ➖ **Минусы / Цена:** data loss risk if master dies before replicate; stale reads from replicas (replication lag).
- 📍 **Где применять:** DR, failover, cross-region standby, analytics replica (read-only). Optional read offload — с lag.

## Semi-sync (компромисс)

- ACK from **at least one** replica → balance durability vs latency.

**Метрика:** replication lag (seconds) — monitor on Deep Dive §4.x observability.


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

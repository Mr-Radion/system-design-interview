---
layer: pattern
steps: [4]
related:
  - constraints/consistency-as-nfr
  - data/master-slave-multi-master
---

# Sync vs Async Replication (синхронная vs асинхронная репликация)

> **Главное:** Replication — sync vs async. Вход — RPO/RTO и availability (шаг 2).

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужен RPO≈0? | Да | Sync replication |
| Можно ли иметь stale reads? | Да | Async replication |

## Цепочка решений

Шаг 2 NFR → шаг 5 replication pattern → шаг 2 Infra tech

## Synchronous Replication

**Выбор:** master ждёт ACK от replica(s) перед commit.

- ➕ **Плюсы:** zero data loss on master fail; replicas always up-to-date; strong consistency reads from replica.
- ➖ **Минусы / Цена:** write latency ↑ (network RTT (Round Trip Time, время кругового пути); one slow replica blocks all writes; availability ↓ on partition.
- 📍 **Где применять:** financial ledger, inventory with strict consistency.

## Asynchronous Replication

**Выбор:** master commits locally, replicas catch up in background.

- ➕ **Плюсы:** minimal write latency; master availability; geo-distributed replicas cheap.
- ➖ **Минусы / Цена:** data loss risk if master dies before replicate; stale reads from replicas (replication lag).
- 📍 **Где применять:** read scaling (read replicas), analytics replica, cross-region DR (Disaster Recovery, аварийное восстановление).

## Semi-sync (компромисс)

- ACK from **at least one** replica → balance durability vs latency.

**Метрика:** replication lag (seconds) — monitor on step 5 observability.


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

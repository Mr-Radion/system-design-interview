---
layer: pattern
steps: [4]
related:
  - data/replication-sync-async
  - constraints/consistency-as-nfr
---

# Master-Slave vs Multi-Master (ведущий-ведомый vs multi-master)

> **Главное:** Master-Slave vs Multi-Master — распределение лидера и запись. Вход — conflict tolerance.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужен single writer? | Да | Master-Slave |
| Need multi-site writes? | Да | Multi-Master (conflict resolution) |

## Цепочка решений

Шаг 2 NFR → шаг 5 replication/topology → шаг 2 Infra tech

## Master-Slave (Primary-Replica)

**Выбор:** один writer (master), N replicas для **HA / DR**.

- ➕ **Плюсы:** failover (promoted replica); DR; optional read offload с lag; простая модель (writes → master).
- ➖ **Минусы / Цена:** write bottleneck on master; replication lag на replica reads; failover = brief downtime или manual.
- 📍 **Где применять:** 90% web apps — **availability**, не primary read scale. Read throughput → cache/CDN.

## Multi-Master (Active-Active)

**Выбор:** несколько nodes принимают writes.

- ➕ **Плюсы:** write scale; geo-write locality; no single write bottleneck.
- ➖ **Минусы / Цена:** write conflicts; conflict resolution (LWW, CRDT); split-brain risk; complex ops.
- 📍 **Где применять:** geo-distributed writes (Cassandra, CouchDB), collaborative docs.

## Read from Primary vs Replica

| Read source | Consistency | Latency |
|-------------|-------------|---------|
| Primary | Strong | Higher load on master |
| Replica | Eventual (lag) | Offloads master |

**Правило:** financial reads → primary. Feed → replica OK.


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

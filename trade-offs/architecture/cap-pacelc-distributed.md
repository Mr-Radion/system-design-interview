---
layer: pattern
steps: [5]
related:
  - constraints/consistency-as-nfr
  - data/replication-sync-async
  - architecture/messaging-patterns
  - api/sync-async-messaging
---

# CAP (Consistency, Availability, Partition tolerance) / PACELC (теоремы распределённых систем)

> **Главное:** CAP/PACELC — это **не выбор технологии**. Это ответ на вопрос «**как система ведёт себя при сбое сети и в normal time**».  
> Вход — **NFR шага 2** ([consistency-as-nfr](../constraints/consistency-as-nfr.md)).  
> Выход — **паттерны шага 5** → **sync/async шага 3** → **технологии шага 6**.

## Что определяет выбор в CAP?

CAP не «выбирается из воздуха». На **шаге 2** вы уже решили по **бизнес-операции**:

| Вопрос на шаге 2 | Если «да» | CAP-профиль |
|------------------|-----------|-------------|
| Потеря 1 ₽ / 1 места в самолёте недопустима? | Strong consistency | **CP** при partition |
| Сервис должен отвечать даже при рассинхроне узлов? | Availability важнее | **AP** при partition |
| Пользователь ждёт ответ < 100 ms, расхождение 1–2 сек OK? | Low latency | **PACELC → EL** |
| Отчёт / баланс должны быть точными всегда? | Consistency в normal time | **PACELC → EC** |

**Partition tolerance (P)** в production **всегда да** — сеть ломается. Поэтому реальный выбор: **C или A при partition**, плюс **L или C без partition** (PACELC).

## Цепочка решений (4 слоя)

```
Шаг 2  NFR          →  «платёж = strong, лайк = eventual»
        ↓
Шаг 5  CAP/PACELC  →  «при partition: CP или AP; иначе: EL или EC»
        ↓
Шаг 5  Паттерны     →  sync/async repl, CQRS, Outbox, Saga, cache…
        ↓
Шаг 3  API          →  sync RPC (платёж) vs async queue (email после заказа)
        ↓
Шаг 2 Infra  Технологии   →  PostgreSQL, Cassandra, Kafka…
```

**CAP влияет в первую очередь на паттерны данных и репликации**, не напрямую на «REST vs gRPC».  
Sync/async API — **отдельный** trade-off ([sync-async-messaging](../api/sync-async-messaging.md)), но **согласован** с CAP:

| CAP/PACELC | Типичные паттерны (шаг 5) | Sync vs Async (шаг 3) | Технологии (шаг 2 Infra) |
|------------|---------------------------|-------------------------|---------------------|
| **CP** | ACID TX, sync replication, quorum, Saga (multi-service) | **Sync** для критичного path; Outbox для side-effects | PostgreSQL + sync replica, Spanner, etcd |
| **AP** | Async replication, cache-aside, denormalized reads | **Async** events, at-least-once + idempotency | Cassandra, DynamoDB, Redis + Kafka |
| **PACELC → EL** | CQRS-lite, read replicas (stale OK), CDN cache | Sync read from cache; async write pipeline | Redis, Cassandra, Kafka, read replicas |
| **PACELC → EC** | Strong read path, sync repl even in normal time | Sync reads/writes on critical path | PostgreSQL, CockroachDB |

### Outbox — мост между Strong DB и Async

Когда нужно **и** strong consistency в БД, **и** async доставка в другие сервисы:

1. В **одной транзакции**: запись в business table + запись в `outbox` table.
2. Отдельный **poller** читает outbox → публикует в Kafka/RabbitMQ.
3. Потребители обрабатывают **async**, с **idempotency**.

→ Паттерн: [messaging-patterns](../architecture/messaging-patterns.md). CAP: **CP для write**, **AP/async для fan-out**.

### CQRS — когда read и write требуют разного CAP-профиля

| Сценарий | Write path | Read path |
|----------|------------|-----------|
| E-commerce заказ | **CP** — strong, ACID | **EL** — лента заказов из read model / cache |
| Social feed | **AP** — async ingest | **EL** — denormalized feed cache |
| Banking | **CP** — ledger | **EC** — баланс всегда точный |

CQRS = **разные модели** под разные NFR, не «всегда eventual».

## CAP Theorem (кратко)

При network **Partition** — только **C** или **A**:

| | CP | AP |
|--|----|----|
| При partition | Consistent, может reject writes | Available, stale reads |
| Примеры | ZooKeeper, etcd, Spanner* | Cassandra, DynamoDB |

*Spanner — особый случай (TrueTime).

## PACELC Extension

**If Partition:** A or C (как CAP).  
**Else (normal):** **L**atency or **C**onsistency.

| Type | Partition | Normal | Example |
|------|-----------|--------|---------|
| PC/EC | Consistency | Consistency | Spanner, CockroachDB |
| PA/EL | Availability | Low Latency | Cassandra, DynamoDB |
| PC/EL | Consistency | Low Latency | MongoDB (default) |

## Практика по типам задач

| Тип задачи | NFR (шаг 2) | CAP/PACELC | Паттерны | API | Технологии |
|------------|-------------|------------|----------|-----|------------|
| **Транзакционные** (платёж, инвентарь, бронь) | Strong, RPO≈0 | CP + EC | ACID, sync repl, Saga, **Transactional Outbox** | Sync RPC/REST | PostgreSQL, sync replica |
| **Менее транзакционные** (лайки, просмотры, feed) | Eventual ± сек | AP + EL | Async repl, cache, CQRS-lite, counter in Redis | Async events | Cassandra, Redis, Kafka |
| **Real-time** (чат, gaming, match) | Low latency + causal | AP + EL | WebSocket, Redis Pub/Sub, local cache | Sync transport + async persist | Redis, Kafka, WS |
| **Analytics / batch** | Throughput > latency | AP + EL | Batch ETL, stream | Async pipeline | Spark, Kafka, S3 — [etl-pipeline-pattern](../architecture/etl-pipeline-pattern.md) |

**Real-time ≠ always strong consistency.** Uber match — low latency (**EL**), eventual OK на карте; payment в поездке — **CP** отдельным sync-сервисом.

## Strong vs Eventual (implementation)

| | Strong | Eventual |
|--|--------|----------|
| Write | Sync replicate / quorum | Async replicate |
| Read | Primary / quorum | Any replica |
| Example | Bank balance | Social likes |

## Consistent Hashing / Quorum / Leader Election

- **Consistent Hashing** — минимальный remap при добавлении нод ([sharding](../data/sharding-partitioning.md)).
- **Quorum R+W>N** — tune consistency vs latency.
- **Leader election** — [distributed-coordination.md](distributed-coordination.md).

---

## Примечания (Habr, часть 1)

### Зачем distributed system

- Workload distribution → больше RPS.
- Geo-replicas → read from nearest node.
- **Node** — один сервер в кластере.

### CAP — пример 3 nodes A, B, C

- **Consistency:** update on B → replicates to A, C before read OK.
- **Availability:** B fails → A, C still serve.
- **Partition:** B isolated → CAP dilemma.

**Источник:** [часть 1](https://habr.com/ru/articles/873388/)

---


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

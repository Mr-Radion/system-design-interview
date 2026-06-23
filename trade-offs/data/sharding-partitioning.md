---
layer: pattern
steps: [4, 5]
related:
  - architecture/cap-pacelc-distributed
  - technologies/databases
---

# Sharding vs Partitioning (шардирование vs партиционирование)

> **Главное:** Sharding/partitioning — масштабирование write/read. Вход — 데이터 scale и access patterns.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужна горизонтальная масштабируемость? | Да | Sharding |
| Geo-local reads important? | Да | Geo-partitioning |

## Цепочка решений

Шаг 2 scale → шаг 4 partition plan → шаг 5 pattern → шаг 2 Infra tech

## Partitioning (в одной БД/ноде)

**Выбор:** разбить таблицу на partitions (by date, range) внутри одного PostgreSQL.

- ➕ **Плюсы:** query pruning (`WHERE date = today` → one partition); easier ops than sharding; ACID (Atomicity, Consistency, Isolation, Durability) preserved.
- ➖ **Минусы / Цена:** still one machine limit; cross-partition queries expensive.
- 📍 **Где применять:** time-series logs, events table > 100GB on single beefy DB.

## Sharding (между нодами)

**Выбор:** распределить data across multiple DB servers by shard key.

- ➕ **Плюсы:** horizontal DB scale; isolate hot tenants.
- ➖ **Минусы / Цена:** no cross-shard JOIN; rebalancing pain; application routing logic; distributed transactions hard.
- 📍 **Где применять:** multi-tenant SaaS, Instagram DMs, Uber trips by city.

## Range-based Sharding

- ➕ **Плюсы:** efficient range queries (`BETWEEN date X AND Y`).
- ➖ **Минусы / Цена:** hotspots (shard «A» overloaded, «Z» empty); manual rebalancing.
- 📍 **Где применять:** time-based data, alphabetical with known distribution.

## Hash-based Sharding

- ➕ **Плюсы:** even distribution; no hotspots.
- ➖ **Минусы / Цена:** no range queries (scatter-gather all shards); resharding → consistent hashing.
- 📍 **Где применять:** user_id sharding, uniform access patterns.

## Geographic / Entity-based Sharding

**Выбор:** шард по региону, tenant, business entity.

- ➕ **Плюсы:** data locality (EU users → EU shard); compliance (GDPR); lower latency for regional queries.
- ➖ **Минусы / Цена:** regional hotspots (all US traffic → US shard overloaded); uneven growth across regions.
- 📍 **Где применять:** multi-region SaaS, Uber by city, EU/US data residency.

## Consistent Hashing (консистентное хеширование)

- При добавлении нод — минимальный data movement vs naive `hash % N`.

## Недостатки sharding (trade-off «делать / не делать»)

| Проблема | Последствие |
|----------|-------------|
| App-level routing | Код знает shard по key; PostgreSQL не роутит за вас (vs partitioning) |
| Cross-shard JOIN | Дорого — fetch from N shards + merge in app |
| Consistency | Транзакции across shards ≈ Saga; no simple ACID (Atomicity, Consistency, Isolation, Durability) |
| Rebalancing | Adding shard → consistent hashing или painful migration |

---

## Примечания (Habr, часть 2)

### Partitioning vs Sharding — ключевая разница

| | Partitioning | Sharding |
|--|--------------|----------|
| Где таблицы | Один сервер БД | Разные серверы |
| Routing | PostgreSQL сам (прозрачно) | Application code |
| Пример SQL | `PARTITION BY RANGE (created_at)` | Shard 1: user_id 1–1000 на DB-1 |

**Запрос `SELECT * FROM users WHERE id=4`** — при partitioning тот же SQL; при sharding app решает: id 4 → DB-1.

### Стратегии sharding (из статьи)

1. **Range:** user_id 1–1000, 1001–2000 — просто, но hotspots на «молодых» id.
2. **Hash:** `HASH(user_id) % N` — равномерно, но rebalance при добавлении shard.
3. **Geographic:** America / Europe shards — latency + compliance, но uneven load.

### Порядок масштабирования (итог Habr)

1. Vertical scaling
2. Indexing → Partitioning
3. Master-Slave (reads)
4. Sharding (writes / size) — **только если всё выше не хватило**

**Over-engineering:** при 10K users не шардируйте под 10M.

**Источник:** [часть 2](https://habr.com/ru/articles/877312/)


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

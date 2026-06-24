---
layer: product
steps: [6]
related:
  - architecture/caching-patterns
---

# Caches (выбор кэша)

> **Главное:** Caches — in-memory caches & tiers. Вход — latency and read amplification.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need sub-ms reads? | Да | In-memory cache |
| Need distributed cache? | Да | Redis Cluster |

## Цепочка решений

Deep Dive §4.x pattern → Deep Dive §4.x (tech)

## Redis (кэш Redis)

- ➕ Rich data structures (hash, sorted set, pub/sub); persistence optional; cluster mode; Lua scripts; TTL (Time To Live, время жизни).
- ➖ Higher memory per key; single-threaded (CPU (Central Processing Unit, процессор) bound on huge ops); ops complexity at cluster scale.
- 📍 **Default** for server-side cache, sessions, rate limiting, leaderboards.

## Memcached (распределённый кэш)

- ➕ Simple; multi-threaded; lower memory overhead; very fast pure KV.
- ➖ No persistence; no data structures; no pub/sub; no replication built-in.
- 📍 **Where:** pure cache layer, simplest KV TTL (Time To Live, время жизни), Facebook-style.

## In-process (local cache)

- ➕ Zero network latency; no infra.
- ➖ Not shared across instances; invalidation nightmare in cluster; memory per pod.
- 📍 **Where:** config, static reference data, Caffeine/Go sync.Map.

## Comparison (сравнение)

| Feature | Redis | Memcached | Local |
|---------|-------|-----------|-------|
| Structures | ✅ | ❌ | ❌ |
| Persistence | ✅ | ❌ | ❌ |
| Pub/Sub (Publish/Subscribe, публикация/подписка) | ✅ | ❌ | ❌ |
| Multi-thread | ❌* | ✅ | ✅ |
| Distributed | ✅ Cluster | Client sharding | ❌ |

*Redis 6+ has I/O threads but single command thread.

**Правило:** Redis unless you need only dumb KV cache at extreme simplicity.

---

## Примечания (Habr, часть 3) — Redis практика

### Почему Redis быстрый, но не заменяет БД

- Data in **RAM (Random Access Memory, оперативная память)** — read/write orders of magnitude faster than disk DB.
- RAM (Random Access Memory, оперативная память) **ограничена** — «Memory limit exceeded» если хранить всё в Redis.
- **Rule:** hot data in Redis, source of truth in DB.

### Key naming convention

```
user:1           — user object
user:2:email     — nested field
```

Разделитель `:` — industry standard.

### Data structures (основные)

| Type | Commands | Use |
|------|----------|-----|
| String | SET, GET, SET NX | Cache, locks |
| List | LPUSH, RPOP | Queue (FIFO (First In First Out, первым пришёл — первым вышел): LPUSH+RPOP), Stack (LPUSH+LPOP) |

`SET key value NX` — set only if not exists (idempotency / dedup).

### Local setup

```bash
docker run -d --name redis-stack -p 6379:6379 redis/redis-stack:latest
```

Полный deep dive — в оригинале серии; на interview достаточно: structures, TTL (Time To Live, время жизни), hit/miss, Pub/Sub (Publish/Subscribe, публикация/подписка) для chat relay.

**Источник:** [часть 3](https://habr.com/ru/articles/885054/)


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

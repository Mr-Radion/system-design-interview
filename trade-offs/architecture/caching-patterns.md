---
layer: pattern
steps: [5]
related:
  - technologies/caches
  - constraints/latency-vs-throughput
---

# Caching Patterns (паттерны кэширования)

> **Главное:** Caching patterns — read/write cache strategies. Вход — read/write ratio и допустимая stale степень.

## Что определяет выбор

| Сигнал | Выбор |
|--------|-------|
| Read-heavy & low-latency | Cache-aside |
| Strong consistency for cache writes | Write-through |
| Burst writes and cache pressure | Write-back |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.2 cache → Deep Dive §4.2 (Redis)

## Cache-Aside (Lazy Loading)

**Flow:** app → cache → miss → DB → populate cache.

- ➕ **Плюсы:** cache only hot data; cache down → DB still works.
- ➖ **Минусы / Цена:** miss = double latency; stale data if invalidation wrong; cache stampede risk.
- 📍 **Где применять:** product catalog, user profiles, feed (most common).

## Write-Through

**Flow:** app → cache → cache sync writes DB.

- ➕ **Плюсы:** cache always consistent with DB on write; fast reads.
- ➖ **Минусы / Цена:** double write latency; cache fills with cold data never read.
- 📍 **Где применять:** read-heavy + consistency critical on read path.

## Write-Back (Write-Behind)

**Flow:** app → cache only → async batch flush to DB.

- ➕ **Плюсы:** blazing write speed; batch DB writes.
- ➖ **Минусы / Цена:** data loss if cache dies before flush; complexity.
- 📍 **Где применять:** like counters, gaming stats, analytics buffers.

## Cache Placement (levels)

| Level | Latency | Invalidation |
|-------|---------|--------------|
| Client-side | Lowest | Hardest |
| CDN (Content Delivery Network, сеть доставки контента) | Low | TTL (Time To Live, время жизни) / purge |
| Server-side (Redis) | Medium | App logic |
| DB query cache | — | Deprecated in most DBs |

## Cache Stampede Mitigation

- Probabilistic early expiration
- Mutex/lock on rebuild
- Pre-warm on deploy

**HTTP / CDN:** кэшировать только **idempotent GET/HEAD**; invalidation — ETag, Cache-Control, versioned keys.

**Deep Dive §4.x:** pattern · eviction policy → [cache-eviction-policies](cache-eviction-policies.md) · Deep Dive §4.x в example.

## Резюме

- Read-heavy + stale OK → **cache-aside** (90% кейсов).
- Read must match DB on write → **write-through**.
- Burst writes, eventual OK → **write-back**.
- Кэш **ускоряет**, не **несёт** нагрузку вместо БД при miss storm.
- Invalidation: TTL + delete-on-write; для catalog — versioned keys.

## FAQ (собес)

| Вопрос | Ответ |
|--------|-------|
| Cache-aside vs write-through? | Aside: populate on miss. Through: cache writes DB sync. |
| Что при cache down? | Aside: fallback DB. Through: writes fail or bypass. |
| Cache stampede? | Lock on rebuild, probabilistic early expiry, pre-warm. |
| Где CDN vs Redis? | CDN — static/media edge. Redis — dynamic hot keys. |
| Patterns vs eviction? | Pattern = read/write flow. Eviction = LRU/2Q при full cache. |

---

## Примечания (Habr, часть 3)

### 4 типа кэша

1. **Client-side** — browser cache (HTML, CSS, JS); меньше запросов к серверу.
2. **Server-side** — Redis/Memcached in-memory.
3. **CDN (Content Delivery Network, сеть доставки контента)** — static assets at edge (CloudFront, Cloudflare).
4. **Application-level** — in-process cache в коде (промежуточные результаты).

### Cache Hit vs Cache Miss

- **Hit:** data in cache → fast (blog example: 800ms DB → 20ms Redis on repeat).
- **Miss:** first request → DB → populate cache.

### Invalidation

- **TTL (Time To Live, время жизни):** e.g. 24h — blogs auto-expire, next request refreshes from DB.
- **On write:** при новом blog post — delete/update cache key.

### Write DB + Write Cache simultaneously

**Codeforces pattern:** on every submission update DB **and** Redis leaderboard → user reads ranking from Redis instantly.

Это variant **Write-Through** — не ждём cache miss для hot read data.

### Blog route example (`/blogs`)

```
1st request → DB 800ms → SET redis blogs:all TTL (Time To Live, время жизни) 24h
2nd request → GET redis → 20ms
New blog posted → DEL blogs:all (or update)
```

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

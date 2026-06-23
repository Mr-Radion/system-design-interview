---
layer: pattern
steps: [5]
related:
  - architecture/caching-patterns
  - technologies/caches
---

# Cache Eviction Policies (политики вытеснения кэша)

> **Главное:** eviction — **что удалить** при full cache. Отдельно от **pattern** (cache-aside / write-through) → [caching-patterns](caching-patterns.md).  
> Вход — размер cache, access pattern, допустимый miss rate.

## Цепочка решений

```
Шаг 2 NFR (latency, read:write) → pattern (aside/through) → eviction policy → Redis/Memcached §6
```

## Формула (back-of-envelope)

```
AvgReadTime = DB_time × miss_rate + cache_time × (1 - miss_rate)
```

Кэш должен **снижать** DB load. Если miss_rate → 100% (cache miss attack) — БД падает.

## Алгоритмы

| Алгоритм | Как работает | Когда |
|----------|--------------|-------|
| **LRU** | выкидываем least recently used | default, general purpose |
| **LFU** | least frequently used | hot keys stable over time |
| **Belady (OPT)** | выкидываем то, что дольше всего не понадобится | теоретический ceiling, не реализуем |
| **Second Chance / Clock** | LRU + reference bit второй шанс | cheap LRU approx |
| **2Q** | separate hot + cold queues | scan-resistant (one-time large scan) |
| **SLRU / TLRU / LRU-k** | segmented / time-aware LRU | large caches, mixed workloads |

## Invalidation (не eviction)

| Паттерн | Пример |
|---------|--------|
| TTL | `EXPIRE key 3600` |
| Delete on write | `DEL feed:{user_id}` при новом посте |
| Versioned key | `icon_v2` — старые ключи протухают сами |
| Tag-based | invalidate all keys with tag `product:42` |

## Anti-patterns

| Ошибка | Почему |
|--------|--------|
| Cache miss attack | random keys → 100% miss → DB overload |
| Huge TTL без invalidation | stale forever |
| Cache larger than working set без eviction tuning | wasted RAM |

## Резюме

- Pattern (aside) решает **где** cache; eviction — **что выкинуть**.
- LRU достаточно на 90% собесов; 2Q — если спросят про scan resistance.
- Считай miss_rate → формула выше.

## FAQ (собес)

| Вопрос | Ответ |
|--------|-------|
| LRU vs LFU? | LRU — recency. LFU — frequency. LFU лучше для stable hot set. |
| Belady? | Optimal offline algorithm; proves LRU can be suboptimal. |
| Redis eviction? | `maxmemory-policy`: allkeys-lru, volatile-lru, etc. |
| Versioned keys vs DEL? | Version — no thundering herd on invalidation. |

→ [caches](../technologies/caches.md) · [GLOSSARY](../../GLOSSARY.md)

---

## Сокращения

| Аббревиатура | Расшифровка |
|--------------|-------------|
| LRU | Least Recently Used |
| LFU | Least Frequently Used |
| SLRU | Segmented LRU |

Полный индекс: [FRAMEWORK.md](../../FRAMEWORK.md#trade-offs).

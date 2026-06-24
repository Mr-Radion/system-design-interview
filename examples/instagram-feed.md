# Пример: Instagram-like feed

← [FRAMEWORK.md](../FRAMEWORK.md) · [instagram-feed.md](instagram-feed.md)

**Overview:** post → async fan-out → feed cache · read bandwidth ~20 GB/s >> write

**50M users · без geo · 1 пост / 5 дней · лента 5×/день · ~700 KB/пост**

---

## 1. FR (5–8 min)

| ID | Требование | Пояснение |
|----|------------|-----------|
| **FR-1** | User загружает пост (текст + 1 фото) | Sync ACK metadata; media — presigned upload |
| **FR-2** | Лента подписок — reverse chrono | Pagination; stale OK (секунды) |
| **FR-3** | Like/unlike **идемпотентен** | Double-click safe |
| **FR-4** | Follow/unfollow — strong consistency | Unfollow сразу убирает из ленты |
| **FR-5** | Celebrity fan-out async | N followers — не sync в POST |
| **FR-6** | Read >> write по bandwidth | ~20 GB/s read vs ~80 MB/s write |

**Out of scope:** DMs, search, geo feed, video transcode

---

## 2. NFR (5–7 min)

### 2.2 Расчёты

| Метрика | Формула | Результат |
|---------|---------|-----------|
| Users | — | **50M** |
| Write QPS | 50M ÷ 5 ÷ 86_400 | **~115** |
| Read QPS | 50M × 5 ÷ 86_400 | **~2_900** |
| Read:Write | | **~25 : 1** |
| Bandwidth read | 2_900 × 10 × 700 KB | **~20 GB/s** |
| Storage / year | 80 MB/s × 86_400 × 365 | **~2.5 TB** |

**Драйвер:** FR-6 — read bandwidth доминирует.

### 2.3 SLA / SLO

| Метрика | Цель |
|---------|------|
| GET feed p50 / p95 / p99 | ~190 ms / ~500 ms / **≤ 2 s** |
| POST post p99 | **≤ 2 s** |
| SLA uptime | **99.9%** |
| SLO | 95% feed requests < 500 ms |
| RPO ленты | секунды (stale OK) · RTO < 15 min |

### 2.4 Throughput

Peak read ~2_900 r/s · write ~115 w/s · burst ×5 prime time · headroom ×2 на CDN.

### 2.5 Observability

| Метрика | Зачем |
|---------|-------|
| `feed_p99_latency_ms` | SLO §2.3 |
| `feed_cache_hit_rate` | cache health |
| `cdn_origin_bandwidth_mbps` | bottleneck alert |

### 2.6 Master Catalog — pillars

| ID | Pillar | ✅ / — | Направление | Почему §2.2/FR | TOP-3? |
|----|--------|--------|-------------|----------------|--------|
| O1 | Availability | ✅ | async repl — HA | SLA 99.9% | — |
| O2 | Continuity | — | — | не спрашивали | — |
| O3 | DR | ✅ | warm tier | RPO сек, RTO 15m | — |
| S1 | Scalability | ✅ | read path 20 GB/s | §2.2 bandwidth | **да** |
| S2 | Consistency | ✅ | strong follow / eventual feed | FR-4, FR-6 | — |
| X1 | Caching | ✅ | CDN + cache-aside | read hot path | **да** |
| X2 | Processing | ✅ | async fan-out | FR-5 | **да** |
| X3 | Observability | ✅ | §2.5 metrics | SLO | — |
| X4 | Security | — | — | out of scope | — |
| X5 | Distributed TX | — | — | no money | — |

### 2.7 Processing paths + DR tier

| Path | Core UC | Когда | Механизм |
|------|---------|-------|----------|
| **Sync** | GET feed, POST follow | user ждёт ответ | API → Redis / PG |
| **Async** | celebrity fan-out | FR-5, N followers | Kafka pub/sub |
| **Batch** | — | — | N/A |

**DR tier (O3):** Warm — RPO секунды, RTO 15 min · async repl standby.

### 2.8 Bottleneck → START §4

**START:** read bandwidth ~20 GB/s → **§4.2** (pillars X1, S1) · **AGENDA:** §3.4 — также X2 → §4.3

---

## 3. HLD (12–15 min)

### 3.1 API

| Endpoint | Зачем | Sync/Async |
|----------|-------|------------|
| `POST /posts` | upload metadata | sync ACK |
| `GET /feed` | лента подписок | sync |
| `POST /follow` | graph edge | sync |

### 3.2 Data

```
User 1──M Post · User M──N User · Post 1──M Like
Store: PostgreSQL (graph) + Object storage (media) + Redis (feed denorm)
```

### 3.3 HLD — схема системы

```mermaid
flowchart TB
    Client --> CDN
    CDN --> ALB["ALB L7"]
    ALB --> Post
    ALB --> Feed
    ALB --> Social
    ALB --> Media

    Post --> Kafka
    Kafka --> FeedWorkers["Feed Workers"]
    FeedWorkers --> Redis["Redis cluster"]

    Media --> S3
    Post --> ShardRouter["Shard Router"]
    ShardRouter --> PGShards["PG 8 shards"]
    Social --> ShardRouter
    Feed --> Redis
    Feed --> PGShards
```

**UC2 лента (data flow):**

```mermaid
flowchart LR
    Client --> Feed
    Feed -->|"hit ~80%"| Redis
    Feed -->|"miss ~20%"| Replica["PG Replica"]
    Client -.->|"фото"| CDN
```

### 3.4 TOP-3 pillars · agenda §4

| # | Pillar (ID) | ✅ Направление | §4 (блок) | Почему |
|---|-------------|----------------|-----------|--------|
| 1 | **X1** Caching | CDN + cache-aside | §4.2 | §2.2 bandwidth |
| 2 | **S1** Scalability | read path 20 GB/s | §4.2 | bottleneck |
| 3 | **X2** Processing | async fan-out | §4.3 | FR-5 |

Implementation: push async, hash shard when needed — §4, не в TOP-3.

---

## 4. Deep Dive (15–18 min) · образец прохода

*На собесе интервьюер выберет **1–2 темы** из START/AGENDA — не все блоки ниже. Это **пример**, как углубиться, если повели в эту сторону.*

**Типичный сценарий:** START §4.2 → по вопросу §4.3 (X2) · §4.4 — только если спросят

### §4.2 DB + Cache *(образец — блок START)*

| Вопрос | ✅ |
|--------|-----|
| SQL vs NoSQL | PostgreSQL — graph + transactions |
| Read hot path | Redis cache-aside feed lists |
| Media bandwidth | CloudFront + S3 |
| HA | async repl — **HA**, stale feed OK |

### §4.3 Broker *(образец — если спросят про X2 / fan-out)*

Kafka pub/sub — celebrity fan-out, replay при lag.

### §4.4 Failures *(pull — 2–3 строки, если осталось время)*

Cache down → PG replica · CDN miss storm → rate limit · Fan-out lag → stale OK.

### Infra sizing

| Компонент | Тех | Размер | Откуда |
|-----------|-----|--------|--------|
| CDN | Cloudflare | ~20 GB/s peak | §2.2 bandwidth |
| Cache | Redis cluster | feed + likes | read-heavy |
| DB | PG 8 shards + 3 repl | metadata | §2.2 storage |
| API | K8s | ~3K r/s | §2.2 read QPS × headroom |
| Broker | Kafka ×3 | fan-out | §2.2 write QPS |

← [FRAMEWORK.md](../FRAMEWORK.md)

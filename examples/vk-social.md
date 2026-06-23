# Пример: VK-like social (capstone)

← [FRAMEWORK.md](../FRAMEWORK.md) · [instagram-feed.md](instagram-feed.md) · [paypal-payments.md](paypal-payments.md)

**Overview:** social graph + feed + messaging · bottleneck = **18.5K msg w/s + 580 TB retention** → message shards first, feed CDN second

**80M DAU · friends + feed + messages + media · messages retention 5 лет**

---

## 1. FR

| ID | Требование | Пояснение |
|----|------------|-----------|
| **FR-1** | Profile + follow / unfollow | Strong consistency на graph; unfollow убирает из feed scope |
| **FR-2** | Feed — посты **только друзей** | Reverse chrono; не global explore |
| **FR-3** | Send message — **sync ACK**, delivery async | Client получает 200 после append; push — optional |
| **FR-4** | Message order **per dialog** | Ordering внутри чата; cross-dialog order не гарантируем |
| **FR-5** | Media attach к post или message | Presigned upload; metadata в PG / message store |
| **FR-6** | Retention messages **5 лет** | Append-only; TTL или partition drop после 5y |
| **FR-7** | At-least-once delivery + **dedup** | Duplicate push не показывает два сообщения client-side |
| **FR-8** | Hot dialog / celebrity messaging | Один dialog_id — высокий write rate; shard hotspot |
| **FR-9** | WebSocket push optional | Pull `GET /messages/{dialog}` always works |
| **FR-10** | Feed fan-out async | Как Instagram FR-5; stale feed OK |
| **FR-11** | **⚠️ Собес:** shard by `dialog_id` vs `user_id` | Locality history read vs balance writes |

### UC → FR

| UC | FR |
|----|-----|
| UC1 Профиль, друзья | FR-1 |
| UC2 Лента постов | FR-2, FR-10 |
| UC3 Личные сообщения | FR-3, FR-4, FR-7, FR-8, FR-9 |
| UC4 Медиа | FR-5, FR-6 |

**Out of scope:** groups, voice/video calls, E2E encryption, global search

**ER:** User M──N User (friends) · User 1──M Post · User 1──M Message · Post 1──M Media

---

## 2. NFR

### 2.1 Входные допущения

| Параметр | Значение |
|----------|----------|
| DAU | 80M |
| Posts | 0.3 / day / active user |
| Feed reads | 8× / day |
| Messages | 20 / day / active user |
| Avg message | 200 B text + 10% with 500 KB media |
| Retention messages | 5 лет |

**Драйвер дизайна:** FR-6 → 580 TB text storage; FR-8 → hash(dialog_id) sharding.

### 2.2 Capacity

| Метрика | Формула | Результат |
|---------|---------|-----------|
| Active users/day | 80M DAU | **80M** |
| Post write RPS | 80M × 0.3 ÷ 86_400 | **~280** |
| Message write RPS | 80M × 20 ÷ 86_400 | **~18_500** |
| Feed read RPS | 80M × 8 ÷ 86_400 | **~7_400** |
| Message storage 5y | 18_500 × 200B × 86_400 × 365 × 5 | **~580 TB** text |
| Media storage 5y | 10% × 500KB × msg volume × 5y | **~ petabyte scale** |

**Вывод:** FR-6 → wide-column message store (§6). FR-8 → dialog_id shard, не user_id alone.

### 2.3 CAP / Consistency

| Участок | Требование | Почему |
|---------|------------|--------|
| friends / profile | strong | FR-1: graph edge immediate |
| feed timeline | eventual OK | FR-10: stale секунды OK |
| messages delivery | at-least-once, order per dialog | FR-4, FR-7: dedup on consumer |

→ [CAP](../trade-offs/architecture/cap-pacelc-distributed.md)

### 2.4 Latency

#### A. Sync — клиент ждёт

**POST /messages (FR-3):**

| Этап | p50 | p99 |
|------|-----|-----|
| API + auth | ~15 ms | ~40 ms |
| Append message shard | ~30 ms | ~120 ms |
| ACK to client | ~5 ms | ~10 ms |
| **Итого send** | **~50 ms** | **≤ 500 ms** |

**GET /feed (FR-2):**

| Этап | p50 | p99 |
|------|-----|-----|
| Cache hit | ~10 ms | ~30 ms |
| Cache miss + assemble | ~200 ms | ~800 ms |
| **Итого feed page** | **~210 ms** | **≤ 1 s** |

#### B. Async — клиент не ждёт

| Процесс | E2E SLO | FR |
|---------|---------|-----|
| Push to recipient WebSocket | ≤ 2 s p95 | FR-9 |
| Fan-out feed on new post | секунды OK | FR-10 |
| Media transcode | минуты OK | FR-5 |

### 2.5 Throughput

Peak message **~18.5K w/s** · feed read **~7.4K r/s** · burst ×3 holidays · message store headroom ×2.

### 2.6 Availability & Failure modes

| Параметр | Значение |
|----------|----------|
| SLA | 99.95% |
| RPO messages | минуты (async repl) |
| RTO | < 30 min |

| Сбой | Поведение | FR |
|------|-----------|-----|
| Hot dialog shard | Rate limit; optional dialog migrate | FR-8 |
| Message queue lag | Delayed push; pull still works | FR-9 |
| Media upload fail | Message text saved; media retry | FR-5 |
| Duplicate delivery | Client dedup by message_id | FR-7 |
| Shard node down | RF=3 read from replica; brief unavailability | FR-6 |

### 2.7 Observability

| Метрика | Зачем | FR / NFR |
|---------|-------|----------|
| `message_send_p99_ms` | Sync ACK SLO | FR-3 |
| `feed_p99_latency_ms` | Feed SLO | FR-2 |
| `message_queue_lag_seconds` | Push delay | FR-9 |
| `dialog_shard_write_rate` | Hot dialog detection | FR-8 |
| `message_store_disk_used_tb` | FR-6 capacity | FR-6 |

### Traceability (FR → NFR → §6)

| FR | NFR driver | Решение в §6 |
|----|------------|--------------|
| FR-6 retention 5y | 580 TB | Scylla / Cassandra, TTL |
| FR-8 hot dialog | write skew | hash(dialog_id) shards |
| FR-10 feed | 7.4K r/s read | Redis cache-aside + Kafka fan-out |
| FR-1 graph | ACID follows | PostgreSQL + read replicas |
| FR-9 push | async delivery | WebSocket gateway |

---

## 3. API

| Вызов | UC | Заметка |
|-------|-----|---------|
| `GET /v1/feed` | UC2 | cursor pagination |
| `POST /v1/friends/{id}` | UC1 | follow |
| `POST /v1/messages` | UC3 | sync ACK, async delivery |
| `GET /v1/messages/{dialog}` | UC3 | pull history |
| `POST /v1/media/upload` | UC4 | presigned upload |

Протокол: **REST** + JSON · real-time optional **WebSocket** для UC3 push → [realtime](../trade-offs/api/realtime-transport.md)

---

## 4. Data

**PostgreSQL** — profiles, friends, posts · **message store** — sharded by dialog/user · **object store** — media · **Redis** — denorm feed lists

### ER — core entities

```mermaid
erDiagram
    USER {
        uuid id PK
        string username
        string display_name
    }
    FOLLOW {
        uuid follower_id FK
        uuid followee_id FK
        timestamp created_at
    }
    POST {
        uuid id PK
        uuid author_id FK
        text body
        timestamp created_at
    }
    DIALOG {
        uuid id PK
        uuid user_a_id FK
        uuid user_b_id FK
    }
    MESSAGE {
        uuid id PK
        uuid dialog_id FK
        uuid sender_id FK
        text body
        timestamp created_at
    }
    MEDIA {
        uuid id PK
        string object_key
        string ref_type
        uuid ref_id FK
    }
    FEED_ITEM {
        uuid viewer_id FK
        uuid post_id FK
        timestamp rank
    }

    USER ||--o{ FOLLOW : follower
    USER ||--o{ FOLLOW : followee
    USER ||--o{ POST : writes
    USER ||--o{ DIALOG : participates
    DIALOG ||--o{ MESSAGE : contains
    USER ||--o{ MESSAGE : sends
    POST ||--o{ MEDIA : attaches
    MESSAGE ||--o| MEDIA : attaches
    USER ||--o{ FEED_ITEM : timeline
    POST ||--o{ FEED_ITEM : in
```

`FOLLOW` — M:N self-ref · `FEED_ITEM` — denorm cache/table для UC2 · `MEDIA.ref_id` → post или message

| Тема | ✅ |
|------|-----|
| SQL graph follows ([sql-nosql](../trade-offs/data/sql-vs-nosql-paradigm.md)) | PostgreSQL |
| Messages volume → wide-column option | Cassandra / Scylla для messages |
| Feed denorm ([norm-denorm](../trade-offs/data/normalization-denormalization.md)) | Redis lists + optional `FEED_ITEM` |

### Размещение по store

```mermaid
flowchart TB
    subgraph pg [PostgreSQL — social core]
        USER
        FOLLOW
        POST
        DIALOG
    end

    subgraph msg [Message Shards — wide-column]
        MESSAGE
    end

    subgraph cache [Redis — denorm]
        FEED_ITEM
    end

    subgraph obj [Object Storage]
        MEDIA
    end

    SocialSvc[Social Service] --> pg
    FeedSvc[Feed Service] --> cache
    FeedSvc --> pg
    MsgSvc[Message Service] --> msg
    MediaSvc[Media Service] --> obj
    MediaSvc --> pg
```

profiles / follows / posts — **ACID** · messages — **append-only, TTL 5y** · media — **blob + metadata in PG**

### Шардирование — hash by dialog_id

```mermaid
flowchart TB
    MsgSvc[Message Service] --> Router["Router<br/>hash dialog_id mod N"]
    Router --> S1[("Shard 1<br/>dialogs A–F")]
    Router --> S2[("Shard 2<br/>dialogs G–M")]
    Router --> SN[("Shard N")]

    Note["Locality: GET /messages/{dialog}<br/>single shard · no scatter-gather"]
    Router -.-> Note
```

`DIALOG` metadata в PG · message body в shard по `dialog_id` → [sharding](../trade-offs/data/sharding-partitioning.md)

### Репликация

```mermaid
flowchart LR
    subgraph social [Social DB]
        PGM[("PG Master")]
        PGR1[("Read Replica 1")]
        PGR2[("Read Replica 2")]
        PGM -->|async| PGR1
        PGM -->|async| PGR2
    end

    subgraph messages [Message store]
        MS1[("Shard primary")]
        MS1r[("Shard replica")]
        MS1 -->|async RF=3| MS1r
    end

    SocialW[Social write] -->|sync profile| PGM
    SocialR[Social read] --> PGR1
    MsgW[Message append] --> MS1
    MsgR[Message history] --> MS1r
```

profile / follows → **sync or strong** · messages → **async RF=3**, RPO минуты OK

### Trade-offs → выбор (data layer)

| Тема | A / B | ✅ Выбор | Почему |
|------|-------|----------|--------|
| Messages store | SQL / wide-column | **wide-column** | 18K w/s, time-range by dialog |
| Social graph | SQL / graph DB | **PostgreSQL** | follows + transactions, scale OK |
| Sharding messages | hash user / dialog | **hash(dialog_id)** | locality per chat |
| Replication | sync / async | **async** messages · **sync** profile | RPO профиля stricter |

---

## 5. HLD

### System context

```mermaid
flowchart TB
    Client --> GW["API Gateway"]
    GW --> Social[Social Service]
    GW --> Feed[Feed Service]
    GW --> Msg[Message Service]
    GW --> Media[Media Service]

    Social --> PG[("PostgreSQL")]
    Feed --> Cache[("Redis")]
    Feed --> PG
    Msg --> MsgStore[("Message Shards")]
    Media --> ObjStore[("Object Storage")]

    PostEvent[Post created] --> Broker[("Event Log")]
    Broker --> FeedWorker[Feed Fan-out]
    FeedWorker --> Cache
```

### UC3 Message flow

```mermaid
sequenceDiagram
    participant C as Client
    participant M as Message Service
    participant S as Message Shard
    participant W as WebSocket

    C->>M: POST /messages
    M->>S: append message
    S-->>M: OK
    M-->>C: 200 ACK
    M->>W: push to recipient
```

---

## 6. Technology choices

### Message store (18K w/s)

| Вопрос | Если да | Если нет |
|--------|---------|----------|
| Write >> read per key? | wide-column | PostgreSQL |
| Time-range by dialog? | partition key dialog_id | — |
| **✅ Выбор** | **Cassandra / Scylla** | append-only, TTL 5y |

### Broker (feed fan-out)

| Вопрос | Если да | Если нет |
|--------|---------|----------|
| Fan-out to N followers? | pub/sub log | queue |
| **✅ Выбор** | **Kafka** | replay, 280 post w/s × followers |

### Cache (feed)

| Вопрос | Выбор |
|--------|-------|
| Hot 15% users = 80% feed reads | Redis cache-aside lists |

### Social graph DB

| Вопрос | Выбор |
|--------|-------|
| ACID follows, joins | PostgreSQL + read replicas |
| Scale | hash(user_id) 4 shards when > single node |

### Infra

| Компонент | Тех | Размер | Откуда |
|-----------|-----|--------|--------|
| Message store | Scylla 6 nodes | ~580 TB+ | §2.2 |
| Social DB | PG 4 shards | profiles, follows | §2.2 posts low |
| Broker | Kafka | fan-out | §2.2 post w/s |
| Cache | Redis | feed hot users | §2.5 read-heavy |
| Object storage | S3 + CDN | media | §2.2 10% media |
| Real-time | WebSocket gateway | UC3 push | §2.4 |
| API | K8s | ~25K combined RPS | §2.5 |

→ [sharding](../trade-offs/data/sharding-partitioning.md) · [messaging](../trade-offs/architecture/messaging-patterns.md) · [cache-eviction](../trade-offs/architecture/cache-eviction-policies.md)

---

← [FRAMEWORK.md](../FRAMEWORK.md)

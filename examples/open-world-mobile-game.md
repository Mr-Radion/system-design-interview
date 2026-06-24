# Пример: mobile open-world game (GTA-like)

← [FRAMEWORK.md](../FRAMEWORK.md) · [instagram-feed.md](instagram-feed.md) · [paypal-payments.md](paypal-payments.md) · [vk-social.md](vk-social.md)

**Overview:** meta-game backend — account · quests · progress · friends · clans · billing · telemetry

*Mobile open-world по мотивам GTA-like; без привязки к конкретному тайтлу.*

---

## 1. FR (5–8 min)

| ID | Требование | Пояснение |
|----|------------|-----------|
| **FR-1** | Account — регистрация, login, link соцсети | OAuth; один аккаунт — несколько providers |
| **FR-2** | Player progress — save/load состояния | Уровень, инвентарь, позиция; не терять при crash |
| **FR-3** | Quests — accept / update / complete | State machine; награды атомарно с progress |
| **FR-4** | Friends + clans | Friend list, invite; clan ≤ 50 members |
| **FR-5** | In-app billing — purchase verify | Webhook store; идемпотентен |
| **FR-6** | Telemetry stream | Игровые события → analytics |

**UC → FR:** UC1 Login → FR-1 · UC2 Save progress → FR-2 · UC3 Complete quest → FR-3 · UC4 Clan invite → FR-4 · UC5 Purchase → FR-5 · UC6 Session events → FR-6

**Акторы:** Player · Mobile Client · Game Server · Auth · Progress · Quest · Social · Billing Service

**Интеграции:** OAuth providers (FR-1) · App Store / Google Play webhook (FR-5) · Analytics pipeline (FR-6)

**Out of scope:** game physics, netcode tick-rate, voice chat, anti-cheat ML, matchmaking PvP ranked

**ER:** Player 1──M QuestProgress · Player M──N Player · Clan 1──M Player

**На собесе уточни:** meta-game API vs dedicated game-server — здесь backend сервисы.

---

## 2. NFR (5–7 min)

### 2.2 Расчёты

**Допущения:** 200M registered · ~3M DAU · peak 20K CCU (из контекста вакансии)

| Метрика | Формула | Результат |
|---------|---------|-----------|
| Registered | из контекста | **200M** |
| DAU | оценка (~1.5% reg) | **~3M** |
| Peak CCU | из вакансии | **20K** |
| Login QPS | 3M × 2 ÷ 86_400 | **~70** avg · **~700** peak |
| Progress save w/s | 20K ÷ 300 s (каждые 5 min) | **~67** |
| Quest event w/s | 20K × 0.3/min ÷ 60 | **~100** |
| Telemetry events/s | 20K × 5/min ÷ 60 | **~1_700** |
| Progress storage | 3M × 50 KB profile | **~150 GB** hot |

**Драйвер:** FR-2/FR-6 — **player state writes** + **telemetry flood** при 20K CCU.

### 2.3 SLA / SLO

| Метрика | Цель |
|---------|------|
| Login p99 | **≤ 500 ms** |
| Progress save p99 | **≤ 300 ms** (sync ACK) |
| Quest complete p99 | **≤ 500 ms** |
| Billing verify p99 | **≤ 2 s** |
| SLA uptime | **99.9%** |
| RPO progress | **≈ 0** (потеря save недопустима) · RTO **< 15 min** |

### 2.4 Throughput

Peak CCU **20K** · login burst ×10 (релиз патча) · telemetry burst ×3 (ивент в зоне) · headroom ×2.

### 2.5 Observability

| Метрика | Зачем |
|---------|-------|
| `ccu_active` | capacity vs 20K target |
| `progress_save_p99_ms` | FR-2 SLO |
| `telemetry_lag_seconds` | ClickHouse pipeline health |
| `billing_webhook_dedup_rate` | FR-5 idempotency |

### 2.6 Master Catalog — pillars

| ID | Pillar | ✅ / — | Направление | Почему §2.2/FR | TOP-3? |
|----|--------|--------|-------------|----------------|--------|
| O1 | Availability | ✅ | multi-AZ, repl — HA | SLA 99.9%, 20K CCU | — |
| O2 | Continuity | ✅ | rolling deploy без downtime | патчи часто | — |
| O3 | DR | ✅ | warm tier | RPO progress ≈ 0 | — |
| S1 | Scalability | ✅ | shard player_id, 20K CCU | §2.2 | **да** |
| S2 | Consistency | ✅ | strong progress/quest | FR-2, FR-3 | **да** |
| X1 | Caching | ✅ | hot profile cache | login, friend list | — |
| X2 | Processing | ✅ | sync save + async telemetry | FR-6, billing | **да** |
| X3 | Observability | ✅ | §2.5 + load tests | нагрузочные тесты | — |
| X4 | Security | ✅ | OAuth, billing signature | FR-1, FR-5 | — |
| X5 | Distributed TX | ✅ | quest reward + progress | FR-3, FR-5 | — |

### 2.7 Processing paths + DR tier

| Path | Core UC | Когда | Механизм |
|------|---------|-------|----------|
| **Sync** | progress save, quest complete, login | user ждёт ACK | API → SQL DB shard |
| **Async** | telemetry, billing webhook, clan notify | FR-5, FR-6 | message bus → workers |
| **Batch** | analytics aggregates, retention | daily reports | analytics warehouse ETL |

**DR tier (O3):** Warm — RPO ≈ 0 на progress (semi-sync/WAL), RTO 15 min.

### 2.8 Bottleneck → куда копать в §4

**Куда копать:** 20K CCU → progress writes + telemetry ~1.7K events/s → Deep Dive **§4.2** (TOP-3: S1, S2, X2 — см. §2.6)

**На собесе акцент (помимо bottleneck):**
- потеря progress при crash — **S2, RPO ≈ 0**
- hot zone / clan event — write spike на один shard
- billing duplicate webhook — **X5 idempotency**
- telemetry не блокирует gameplay — **async path**

---

## 3. HLD (12–15 min)

### 3.1 API

| Endpoint | Зачем | Sync/Async |
|----------|-------|------------|
| `POST /v1/auth/login` | login + OAuth link | sync |
| `PUT /v1/players/{id}/progress` | checkpoint save | sync ACK |
| `POST /v1/quests/{id}/complete` | quest + reward | sync |
| `GET /v1/friends` | friend list | sync |
| `POST /v1/billing/webhook` | store receipt | sync verify, async grant |
| `POST /v1/events` | client telemetry batch | async ACK 202 |

### 3.2 Data

```
Player 1──M QuestProgress · Player M──N Player · Clan 1──M Player  *(ER — §1)*
Store roles: SQL DB (progress, social, quests) · Cache (session, profile) · Message queue (events) · Analytics store
```

### 3.3 HLD — схема системы

```mermaid
flowchart TB
    Mobile --> GW["API Gateway L7"]
    GW --> Auth[Auth Service]
    GW --> Progress[Progress Service]
    GW --> Quest[Quest Service]
    GW --> Social[Social Service]
    GW --> Billing[Billing Service]

    Auth --> PG_Auth[("PostgreSQL")]
    Progress --> ShardRouter["Shard Router"]
    ShardRouter --> PG_P1[("Player Shard 1")]
    ShardRouter --> PG_Pn[("Player Shard N")]
    Progress --> Redis[("Redis")]

    Quest --> ShardRouter
    Social --> PG_Social[("PostgreSQL")]
    Billing --> PG_Bill[("PostgreSQL")]

    Progress --> MQ[("Message Queue")]
    Quest --> MQ
    Billing --> MQ

    MQ --> TelemetryWorker[Telemetry Worker]
    MQ --> BillingWorker[Billing Worker]
    TelemetryWorker --> Analytics[("Analytics Store")]
    BillingWorker --> ShardRouter

    GameServer["Game Server Fleet"] -.->|"periodic save"| Progress
```

**UC: progress save (data flow):**

```mermaid
sequenceDiagram
    participant GS as Game Server
    participant P as Progress Service
    participant R as Redis
    participant DB as PG Shard

    GS->>P: PUT /progress
    P->>R: invalidate cache
    P->>DB: UPSERT player_state
    DB-->>P: OK
    P-->>GS: 200 ACK
    P->>MQ: PlayerSaved event async
```

---

## 4. Deep Dive (15–18 min) · образец прохода

*Интервьюер выберет **1–2 темы** из 20K CCU / progress / billing. Не проходить все §4 подряд.*

**Типичный сценарий:** §4.2 · §4.4 или §4.3 — **по вопросу интервьюера**

### §4.2 DB + player state *(образец — единственный блок на доске)*

| Вопрос | ✅ |
|--------|-----|
| SQL vs NoSQL | **PostgreSQL** — transactions quest+reward, JSONB inventory |
| Shard key | **hash(`player_id`)** mod N; не geo (mobile open-world) |
| Hot profile | Redis cache-aside TTL 5 min; write-through на save |
| Hot zone spike | rate limit saves/player; queue overflow → client retry |
| HA | async repl per shard — **HA**; progress read primary on save path |

**Pull (если спросят):** S2/RPO progress consistency · X2 telemetry/billing async · edge security/rate limit · infra sizing — таблица ниже

### Infra sizing *(pull, ~2 min)*

| Компонент | Тех | Размер | Откуда |
|-----------|-----|--------|--------|
| API | Node.js K8s | ~2K RPS headroom | §2.2 login+quest peak |
| Player DB | PostgreSQL 16 shards | ~150 GB + growth | §2.2 storage |
| Cache | Redis cluster | hot 20K CCU profiles | §2.2 CCU |
| Broker | Kafka ×3 | ~2K msg/s telemetry | §2.2 events/s |
| Analytics | ClickHouse | telemetry 90d | batch from Kafka |
| Game servers | dedicated fleet | 20K CCU | out of API scope |

---

← [FRAMEWORK.md](../FRAMEWORK.md)

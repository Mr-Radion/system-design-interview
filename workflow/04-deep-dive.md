# Шаг 4 — Deep Dive + Tech (15–18 min)

← [FRAMEWORK](../FRAMEWORK.md)

> **Правило шага 4:**
> 1. Открой с §4.x из **§2.8 (START)**
> 2. Пройди pillars из **§3.4 (AGENDA)** — сначала блок START, затем остальные TOP-3
> 3. Остальные ✅ из §2.6 — только по запросу / если время
>
> **Пример Instagram:** START §4.2 (read bottleneck) → затем §4.3 для X2 из agenda.

4 блока — **меню** по pillar IDs:

| §4 | Pillar IDs | Implementation |
|----|------------|----------------|
| §4.1 | O2, X4 | gateway, rate limit, auth |
| §4.2 | S1, X1, O1 | cache-aside, shard, CDN, repl |
| §4.3 | X2, X5 | messaging, saga, **batch** |
| §4.4 | O3, S2 | DR tier, backup, CAP, failover |

## Routing table

| Bottleneck §2.8 (START) | Start here | Пример |
|-------------------------|------------|--------|
| Read >> write, bandwidth | **§4.2** | Instagram |
| Write fan-out / async | **§4.3** | Instagram celebrity |
| Storage TB, retention | **§4.2** | VK messages |
| CP / RPO ≈ 0 | **§4.4** → **§4.2** | PayPal |
| analytics / ETL FR | **§4.3** batch | reports |
| security / routing | **§4.1** | по запросу |

**Имя продукта (Kafka, Redis) — только после trade-off gate**, с привязкой к §2.2.

---

## 4.1 Edge: LB + Gateway (O2, X4)

| Вопрос | Trade-off | Tech |
|--------|-----------|------|
| L4 vs L7? | [load-balancing-l4-l7](../trade-offs/architecture/load-balancing-l4-l7.md) | ALB / NGINX |
| Rate limit? | [resilience](../trade-offs/architecture/resilience-backpressure.md) | Kong / Envoy |
| Deploy continuity? | [deployment](../trade-offs/architecture/deployment-release-strategies.md) | rolling / blue-green |

---

## 4.2 Data: DB + Cache (S1, X1, O1)

| Вопрос | Trade-off | Tech |
|--------|-----------|------|
| SQL vs NoSQL? | [sql-nosql](../trade-offs/data/sql-vs-nosql-paradigm.md) | PostgreSQL / Scylla |
| Read hot path? | [caching-patterns](../trade-offs/architecture/caching-patterns.md) | cache-aside |
| Write scale? | [sharding](../trade-offs/data/sharding-partitioning.md) | hash shard |
| Media bandwidth? | [cdn-object-storage](../trade-offs/architecture/cdn-object-storage-pattern.md) | CDN + S3 |
| HA? | [replication](../trade-offs/data/replication-sync-async.md) | async repl — **HA** |

**По запросу:** [indexing-strategy](../trade-offs/data/indexing-strategy.md)

---

## 4.3 Async, Messaging & Batch (X2, X5)

| Вопрос | Trade-off | Tech |
|--------|-----------|------|
| Fan-out / decouple? | [messaging](../trade-offs/architecture/messaging-patterns.md) | Kafka pub/sub |
| Task queue vs log? | [brokers](../trade-offs/technologies/message-brokers.md) | Kafka vs RabbitMQ |
| Distributed TX? | [saga-outbox](../trade-offs/architecture/saga-vs-outbox.md) | outbox + orchestrator |
| Reports / archive? | [batch-vs-stream](../trade-offs/architecture/batch-vs-stream.md) | cron / Airflow / ETL |

**Sync vs Async vs Batch** — решение в §2.7; здесь только implementation.

---

## 4.4 DR, CAP & Failures (O3, S2)

| Тема | Trade-off |
|------|-----------|
| CAP по участку | [cap-pacelc](../trade-offs/architecture/cap-pacelc-distributed.md) |
| RPO/RTO | [availability-slo](../trade-offs/constraints/availability-slo-rpo-rto.md) |
| DR pattern | [disaster-recovery](../trade-offs/architecture/disaster-recovery-pattern.md) |

**DR cheat sheet (после tier §2.7):**

| Tier | Backup | Standby | Failover |
|------|--------|---------|----------|
| Hot | WAL / continuous | sync repl | auto (Patroni, etc.) |
| Warm | incremental | async repl | manual / semi-auto |
| Cold | full + incr | none | restore from backup |

Failure modes — 2–3 строки на доске.

---

## Infra sizing (если спросят / 2–3 min)

| Компонент | Тех | Размер | Откуда |
|-----------|-----|--------|--------|
| CDN | … | ~X GB/s | §2.2 bandwidth |
| Cache | … | hot users | §2.2 read QPS |
| DB | … | shards + repl | §2.2 |
| API | … | ~N r/s | §2.2 × headroom |
| Broker | … | fan-out | §2.2 write QPS |

---

← [03 — HLD](03-high-level-design.md) · [FRAMEWORK](../FRAMEWORK.md)

Примеры: [instagram §4](../examples/instagram-feed.md#4-deep-dive) · [paypal §4](../examples/paypal-payments.md#4-deep-dive) · [vk §4](../examples/vk-social.md#4-deep-dive)

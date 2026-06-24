# Шаг 4 — Deep Dive + Tech (15–18 min) · меню, не чеклист

← [FRAMEWORK](../FRAMEWORK.md)

**Фокус шага:** интервьюер **сам выбирает**, куда копать. Обычно **1 блок по bottleneck (START)** + **ещё 0–1 из AGENDA** — не все §4.1–4.4 подряд.

| Сколько | Что | Кто решает |
|---------|-----|------------|
| **1 блок** | START из §2.8 | ты предлагаешь по bottleneck |
| **+0–1 блок** | второй pillar из §3.4 AGENDA | интервьюер ведёт вопросами |
| **остальное** | строки из меню ниже | только если спросил / 2–3 min в конце |

> **Правило шага 4:**
> 1. Предложи **START** — §4.x из §2.8 («bottleneck → начну отсюда»)
> 2. Если интервьюер согласен — **углубись в 1 блок** (2–4 trade-off вопроса, не все строки таблицы)
> 3. **AGENDA (§3.4)** — второй блок только если он повёл туда или осталось ~5 min
> 4. §4.1–4.4 ниже — **меню pull**, не обязательный маршрут

**Мини-примеры прохода** (в [examples/](../examples/) §4) — образец *как* раскрыть 1–2 блока, не план «пройти всё».

| Пример | Обычно на собесе | Второй блок (если успеют) |
|--------|------------------|---------------------------|
| Instagram | §4.2 START | §4.3 по вопросу про fan-out |
| PayPal | §4.4 → §4.2 START | §4.3 saga — если спросят |
| VK | §4.2 START | §4.3 или §4.4 — по вопросу |

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

## 4.1 Edge: LB + Gateway (O2, X4) · pull

*Раскрывать только если интервьюер спросил про gateway / security / deploy.*

| Вопрос | Trade-off | Tech |
|--------|-----------|------|
| L4 vs L7? | [load-balancing-l4-l7](../trade-offs/architecture/load-balancing-l4-l7.md) | ALB / NGINX |
| Rate limit? | [resilience](../trade-offs/architecture/resilience-backpressure.md) | Kong / Envoy |
| Deploy continuity? | [deployment](../trade-offs/architecture/deployment-release-strategies.md) | rolling / blue-green |

---

## 4.2 Data: DB + Cache (S1, X1, O1) · частый START

*Типичный первый блок: read/storage bottleneck. 2–3 строки таблицы, не всю.*

| Вопрос | Trade-off | Tech |
|--------|-----------|------|
| SQL vs NoSQL? | [sql-nosql](../trade-offs/data/sql-vs-nosql-paradigm.md) | PostgreSQL / Scylla |
| Read hot path? | [caching-patterns](../trade-offs/architecture/caching-patterns.md) | cache-aside |
| Write scale? | [sharding](../trade-offs/data/sharding-partitioning.md) | hash shard |
| Media bandwidth? | [cdn-object-storage](../trade-offs/architecture/cdn-object-storage-pattern.md) | CDN + S3 |
| HA? | [replication](../trade-offs/data/replication-sync-async.md) | async repl — **HA** |

**По запросу:** [indexing-strategy](../trade-offs/data/indexing-strategy.md)

---

## 4.3 Async, Messaging & Batch (X2, X5) · второй блок или pull

*Если не START — обычно по вопросу про fan-out, billing async, ETL.*

| Вопрос | Trade-off | Tech |
|--------|-----------|------|
| Fan-out / decouple? | [messaging](../trade-offs/architecture/messaging-patterns.md) | Kafka pub/sub |
| Task queue vs log? | [brokers](../trade-offs/technologies/message-brokers.md) | Kafka vs RabbitMQ |
| Distributed TX? | [saga-outbox](../trade-offs/architecture/saga-vs-outbox.md) | outbox + orchestrator |
| Reports / archive? | [batch-vs-stream](../trade-offs/architecture/batch-vs-stream.md) | cron / Airflow / ETL |

**Sync vs Async vs Batch** — решение в §2.7; здесь только implementation.

---

## 4.4 DR, CAP & Failures (O3, S2) · START для CP/money

*PayPal-стиль: часто открывают здесь. Failure modes — 2–3 строки, не лекция.*

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

Примеры: [instagram §4](../examples/instagram-feed.md#4-deep-dive) · [paypal §4](../examples/paypal-payments.md#4-deep-dive) · [vk §4](../examples/vk-social.md#4-deep-dive) · [open-world §4](../examples/open-world-mobile-game.md#4-deep-dive)

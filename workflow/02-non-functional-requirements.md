# Шаг 2 — NFR (5–7 min на доске)

← [FRAMEWORK](../FRAMEWORK.md)

**Фокус шага:** цифры, SLO, pillars — *без* trade-off решений и вендоров (→ шаг 4).

**На доске за 5–7 min:** §2.0 ключевые ответы · §2.2 цифры + допущения · §2.8 куда копать в §4 одной строкой.  
**В заметках / если успеваешь:** §2.3–2.5 детально · полный pass §2.6–2.7 pillars.

**Три слоя:** A метрики (§2.0–2.5) · **B pillars** (§2.6–2.7) · C implementation (§4).

## 2.0 Спроси интервьюера (перед цифрами)

*Перенесено из шага 1 — это вход для расчётов и pillars, не функциональные требования.*

| Вопрос | Зачем | Куда в workflow |
|--------|-------|-----------------|
| **Scale?** DAU / MAU / CCU / registered? | вход §2.2 | расчёты RPS |
| **Geo?** single region / multi-region? | CDN, repl, latency | §2.6 O3, X1 |
| **Read vs write?** ratio, hot path? | bottleneck | §2.8 |
| **Consistency?** money / social / stale OK? | CAP, RPO | §2.6 S2, §2.3 RPO |
| **Latency?** p99 target на sync path? | SLO | §2.3 |
| **Retention?** how long store data? | storage / year | §2.2 |
| **Peak vs average?** burst events? | headroom | §2.4 |
| **SLA / RPO / RTO?** CP или eventual OK? | DR tier | §2.6 O3, §2.7 |
| **Existing constraints?** stack, compliance? | pull | §4 по запросу |

Если интервьюер молчит — назови **допущение** вслух и запиши в §2.2.

### Связь с шагом 4

§2.6–§2.8 **не решают** trade-offs — они задают **scope** для Deep Dive.

| Секция | Роль | Уровень | Что на доске | Чего не пишем |
|--------|------|---------|--------------|---------------|
| §2.0–2.5 | Метрики | цифры | RPS, SLA, RPO | имена продуктов |
| §2.6 | **Темы архитектуры** + TOP-3 | pillar | ✅/— + 3 главные темы | trade-off имена |
| §2.8 | **Куда копать в §4** | entry point | 1 строка bottleneck → §4.x | детали trade-offs |
| §4 | **Implementation** | trade-off | gate → tech name | новые темы вне §2.6 |

## 2.1 Ключевые метрики

| Метрика | Зачем |
|---------|-------|
| **RPS / QPS** | пропускная способность API / backend |
| **Latency p50 / p95 / p99** | UX; sync path |
| **Объём данных** | DB, трафик, логи, media |
| **Users (DAU / MAU)** | вход для моделирования нагрузки |
| **Частота действий** | reads/day, writes/day на user |
| **SLA** | uptime (99.9%, 99.99%) |
| **SLO** | SRE-цели (95% requests < X ms) |

## 2.2 Предварительные расчёты

| Метрика | Формула | Результат |
|---------|---------|-----------|
| DAU / Users | от интервьюера | … |
| Read QPS | DAU × reads/day ÷ 86_400 | … |
| Write QPS | DAU × writes/day ÷ 86_400 | … |
| Read:Write ratio | | … : 1 |
| Storage / day | writes × row size | … |
| Storage / year | × 365 | … |
| Bandwidth | QPS × payload | … |

```
Read RPS   = users × reads/day ÷ 86_400
Write RPS  = users × writes/day ÷ 86_400
Read Mbps  = read_RPS × items × size
Write Mbps = write_RPS × size
Storage/год = write_Mbps × 86_400 × 365
```

**Драйвер дизайна:** bottleneck → FR-ID → §2.6 pillars (TOP-3) → §4.

## 2.3 SLA / SLO / Latency

| Метрика | Цель | Примечание |
|---------|------|------------|
| Latency p50 / p95 / p99 | … | sync path |
| SLA uptime | … | / month |
| SLO | 95% requests < … ms | SRE |
| RPO / RTO | … | если CP / money → O3 DR |

**Latency breakdown (1–2 sync path, кратко):**

| Этап | p50 | p99 |
|------|-----|-----|
| … | … | … |
| **Итого** | … | **≤ SLO** |

## 2.4 Throughput

Peak … · burst ×N · headroom (из §2.2).

## 2.5 Observability (pillar X3)

| Метрика | Зачем | FR |
|---------|-------|-----|
| `metric_name` | SLO / bottleneck alert | FR-… |

2–4 метрики, связанные с §2.3 SLO.

## 2.6 Master Catalog — pillars · scoping для §4

*Уровень: pillar. Фиксируем что релевантно; детали trade-offs — только в §4.*

**Без имён продуктов.** Отметь **ровно 3** строки `TOP-3? = да` — главные темы для Deep Dive.

| ID | Pillar | ✅ / — | Направление (без имён) | Почему §2.2/FR | TOP-3? |
|----|--------|--------|------------------------|----------------|--------|
| **O1** | Availability | | repl для HA, SLA | §2.3 | |
| **O2** | Continuity | | zero-downtime deploy? | SLA | |
| **O3** | DR | | RPO/RTO; tier §2.7 | §2.3 | |
| **S1** | Scalability | | read / write / storage | §2.2 | |
| **S2** | Consistency (CAP) | | strong / eventual | FR | |
| **X1** | Caching | | CDN / app / DB? | latency, bandwidth | |
| **X2** | Processing model | | sync/async/batch §2.7 | FR paths | |
| **X3** | Observability | | метрики §2.5 | SLO | |
| **X4** | Security / Auth | | JWT, rate limit, PCI? | FR | |
| **X5** | Distributed TX | | saga / 2PC / none | money, cross-shard | |

**Pull (не в pass):** Extensibility, Maintainability, Portability → [FRAMEWORK](../FRAMEWORK.md#pull-on-demand).

**Ранжирование TOP-3:** bottleneck §2.8 → SLA/RPO (O3, S2) → FR (X2 fan-out) → impact §2.2.

### Типичные TOP-3 по типу задачи

| Тип | Pillar IDs |
|-----|------------|
| Read-heavy | **X1** · **S1** · **X2** |
| CP / money | **O3** · **S2** · **X5** |
| Write-heavy | **S1** · **O3** · **X2** |
| Game backend | **S1** · **S2** · **X2** |

### Правило репликации

**Replication = HA/DR (O1, O3), не read scale.** Read throughput → X1/S1 (CDN, cache).

## 2.7 Processing + DR tier · под-выборы для §4

*Уровень: pillar sub-choice (sync/async/batch, hot/warm/cold). Backup, failover, Kafka — §4.*

### Processing model (parent: X2)

| Path | Core UC | Когда | Механизм |
|------|---------|-------|----------|
| **Sync** | … | user ждёт ответ | API → DB |
| **Async** | … | fan-out, decouple, spikes | queue / pub-sub |
| **Batch** | … | reports, archive, не realtime | cron / ETL |

→ [messaging](../trade-offs/architecture/messaging-patterns.md) · [batch-vs-stream](../trade-offs/architecture/batch-vs-stream.md)

### DR tier (parent: O3)

| Tier | RPO | RTO | Практики (§4.4 detail) |
|------|-----|-----|------------------------|
| Hot | сек–мин | мин | sync/semi-sync repl, auto failover |
| Warm | мин–ч | ч | async repl, standby |
| Cold | ч–дни | дни | backups, restore test |

Failover, chaos, multi-region — §4.4, не на доске в 5 мин.

## 2.8 Bottleneck → куда копать в §4

**Bottleneck** = что сломается первым (из §2.2). **Одна строка:** куда начать Deep Dive.

| Если bottleneck… | Начать с | Связанные pillars (§2.6) |
|------------------|----------|--------------------------|
| read bandwidth / latency | **§4.2** | X1, S1 |
| write fan-out / async | **§4.3** | X2 |
| storage / retention | **§4.2** | S1 |
| CP / RPO ≈ 0 | **§4.4** → **§4.2** | O3, S2 |
| security / routing | **§4.1** | X4 |
| analytics / ETL FR | **§4.3** batch | X2 |

**Пример Instagram:** read 20 GB/s → начать **§4.2** · TOP-3 из §2.6: X1, S1, X2 · fan-out (X2) — только если спросят.

**Вывод:** `bottleneck → §4.x` · **3 главные темы** = строки `TOP-3? = да` в §2.6

## Latency — порядок величин (cheat sheet)

| Store / hop | ~latency |
|-------------|----------|
| RAM / L1 cache | µs |
| SSD | ~1 ms |
| HDD | ~10 ms |
| Same DC network | ~0.5 ms |
| Cross-region | ~50–150 ms |

---

← [01 — FR](01-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · [03 — HLD](03-high-level-design.md) →

Примеры: [instagram §2](../examples/instagram-feed.md#2-nfr) · [paypal §2](../examples/paypal-payments.md#2-nfr) · [vk §2](../examples/vk-social.md#2-nfr) · [open-world §2](../examples/open-world-mobile-game.md#2-nfr) · [nutrition §2](../examples/nutrition-mobile-app.md#2-nfr) · [bulk-messaging §2](../examples/bulk-messaging-platform.md#2-nfr)

# Шаг 2 — NFR (5–7 min на доске)

← [FRAMEWORK](../FRAMEWORK.md)

**Фокус шага:** цифры, SLO, pillars — *без* trade-off решений и вендоров (→ шаг 4).

**На доске за 5–7 min:** §2.1 цифры + допущения · §2.2 pillars (TOP-3) + одна строка вывода.

**Два слоя:** A метрики (§2.1) · B pillars + вывод (§2.2) · C implementation (§4).

## 2.1 Цифры на доску

**Расшифровки параметров:** [GLOSSARY §2.1](GLOSSARY.md#step2-metrics) · [NFR / SLA / SLO](GLOSSARY.md#nfr)

Сначала **спроси интервьюера** (если молчит — назови допущение вслух и запиши в таблицу):

| Вопрос | Зачем |
|--------|-------|
| **Scale?** [DAU](GLOSSARY.md#dau) / [MAU](GLOSSARY.md#mau) / [CCU](GLOSSARY.md#ccu) / registered? | посчитать [QPS](GLOSSARY.md#qps) — сколько запросов/сек |
| **Geo?** single region / multi-region? | нужен ли CDN, [repl](GLOSSARY.md#repl), cross-region latency |
| **Read vs write?** ratio, [hot path](GLOSSARY.md#hot-path)? | найти [bottleneck](GLOSSARY.md#bottleneck) — что сломается первым |
| **Consistency?** money / social / stale OK? | strong vs eventual, [RPO](GLOSSARY.md#rpo) |
| **Latency?** [p99](GLOSSARY.md#p99) target на [sync path](GLOSSARY.md#sync-path)? | [SLO](GLOSSARY.md#slo) — цель задержки для UX |
| **Retention?** how long store data? | [storage / year](GLOSSARY.md#formulas) — объём диска |
| **Peak vs average?** burst events? | [headroom](GLOSSARY.md#headroom) в QPS — запас на пик |
| **SLA / RPO / RTO?** CP или eventual OK? | [SLA](GLOSSARY.md#sla) uptime + DR tier ([RPO](GLOSSARY.md#rpo) / [RTO](GLOSSARY.md#rto)) |
| **Existing constraints?** stack, compliance? | pull → §4 |

| Вопрос | Формула / допущение | Результат | На доске |
|--------|---------------------|-----------|----------|
| DAU / Users | от интервьюера | … | **…** users |
| Users online | DAU × 10% (или CCU) | … | ~… concurrent |
| Read QPS | users × reads/day ÷ 86_400 | … | ~… read/s |
| Write QPS | users × writes/day ÷ 86_400 | … | ~… write/s |
| Read:Write | | … : 1 | … : 1 |
| Peak QPS | avg × burst (×3–×5) | … | ~… peak |
| Storage / year | writes × row size × 365 | … | ~… TB |
| Bandwidth | QPS × payload | … | ~… GB/s |
| SLA p99 sync | N hops × SSD ~1 ms + DC ~0.5 ms | … | p99 < … ms |
| SLA uptime | product requirement | … | …% |
| RPO / RTO | CP / money → ≈ 0 | … | RPO … · RTO … |

**Справочник latency** (для back-of-envelope, не на доску):

| Store / hop | ~latency |
|-------------|----------|
| RAM / L1 cache | µs |
| SSD | ~1 ms |
| HDD | ~10 ms |
| Same DC network | ~0.5 ms |
| Cross-region | ~50–150 ms |

**Драйвер дизайна:** bottleneck → FR-ID → §2.2 pillars (TOP-3) → §4.

## 2.2 Pillars + вывод

*Уровень: pillar. Фиксируем что релевантно; trade-offs и tech names — только в §4.*

**Без имён продуктов.** Отметь **ровно 3** строки `TOP-3` в колонке «На доске».

| ID | Pillar | Что спросят | На доске | типично для |
|----|--------|-------------|----------|-------------|
| **O1** | Availability | repl для HA, SLA | ✅ / — | — |
| **O2** | Continuity | zero-downtime deploy? | ✅ / — | — |
| **O3** | DR | RPO/RTO; tier → §4.4 | ✅ / — | CP/money, write-heavy |
| **S1** | Scalability | read / write / storage | ✅ / — | read-heavy, write-heavy, game, messaging |
| **S2** | Consistency (CAP) | strong / eventual | ✅ / — | CP/money, game |
| **X1** | Caching | CDN / app / DB? | ✅ / — | read-heavy, mobile BFF |
| **X2** | Processing model | sync/async/batch → §4.3 | ✅ / — | read-heavy, write-heavy, game, messaging, mobile |
| **X3** | Observability | метрики → §4 pull | ✅ / — | — |
| **X4** | Security / Auth | JWT, rate limit, PCI? | ✅ / — | payments, webhooks |
| **X5** | Distributed TX | saga / 2PC / none → §4.3 | ✅ / — | CP/money, messaging, billing |

**Pull (не в pass):** Extensibility, Maintainability, Portability → [FRAMEWORK](../FRAMEWORK.md#pull-on-demand).

**Ранжирование TOP-3:** bottleneck → SLA/RPO (O3, S2) → FR (X2 fan-out) → impact §2.1.

**Правило репликации:** Replication = HA/DR (O1, O3), не read scale. Read throughput → X1/S1 (CDN, cache).

**Вывод:** bottleneck → **§4.x** · **TOP-3:** … · … · …

---

← [01 — FR](01-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · [03 — HLD](03-high-level-design.md) →

Примеры: [instagram §2](../examples/instagram-feed.md#2-nfr) · [paypal §2](../examples/paypal-payments.md#2-nfr) · [vk §2](../examples/vk-social.md#2-nfr) · [open-world §2](../examples/open-world-mobile-game.md#2-nfr) · [nutrition §2](../examples/nutrition-mobile-app.md#2-nfr) · [bulk-messaging §2](../examples/bulk-messaging-platform.md#2-nfr)

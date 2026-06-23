# Шаг 2 — NFR (цифры и SLO)

← [FRAMEWORK](../FRAMEWORK.md)

## Что фиксируем

**Только цифры и SLO** — без имён технологий, без trade-off решений. Паттерны → [шаг 5](05-architectural-characteristics.md). Infra → [шаг 7](07-technology-choices.md).

## Шаблон §2

### 2.1 Входные допущения

| Параметр | Значение |
|----------|----------|
| … | … |

**Драйвер дизайна:** bottleneck → FR-ID (решение в §5).

### 2.2 Предварительные расчёты (Highload)

| Метрика | Допущение | Формула | Результат | FR |
|---------|-----------|---------|-----------|-----|
| **Пользователи** | … | — | … | — |
| **Частота** | reads/day, writes/day | — | — | FR-… |
| **RPS read** | users × reads ÷ 86_400 | … | **…** | |
| **RPS write** | users × writes ÷ 86_400 | … | **…** | |
| **Объём данных** | row size × volume | … | **…** | |
| **Bandwidth read** | read × items × size | … | **…** | bottleneck? |

**Вывод:** … → §5 / §7.

### 2.3 SLO и целевые метрики

| Метрика | Цель | Примечание |
|---------|------|------------|
| Latency p50 / p95 / p99 | … | sync path |
| SLA uptime | … | / month |
| SLO | 95% requests < … ms | SRE |
| RPO / RTO | … | для §5 DR |

**Latency breakdown (sync):**

| Этап | p50 | p99 |
|------|-----|-----|
| … | … | … |
| **Итого** | … | **≤ SLO** |

**Async (клиент не ждёт):**

| Процесс | E2E SLO | FR |
|---------|---------|-----|

### 2.4 Throughput

Peak … · burst ×N · headroom (из §2.2).

### 2.5 Observability

| Метрика | Зачем | FR |
|---------|-------|-----|
| `metric_name` | … | FR-… |

## Расчёты

```
Read RPS   = users × reads/day ÷ 86_400
Write RPS  = users × writes/day ÷ 86_400
Read Mbps  = read_RPS × постов_в_ответе × размер_поста
Write Mbps = write_RPS × размер_поста
Storage/год = write_Mbps × 86_400 × 365
```

## Latency — порядок величин (back-of-envelope)

| Store / hop | ~latency |
|-------------|----------|
| RAM / L1 cache | µs |
| SSD | ~1 ms |
| HDD | ~10 ms |
| Same DC network | ~0.5 ms |
| Cross-region | ~50–150 ms |

## Чеклист NFR → шаг 5

| Блок §2 | Шаг 5 (архитектура) | Trade-offs |
|---------|----------------------|------------|
| RPO/RTO, SLA | Availability, DR | [SLO/RPO](../trade-offs/constraints/availability-slo-rpo-rto.md) · [DR](../trade-offs/architecture/disaster-recovery-pattern.md) |
| RPS, bandwidth | Scalability | [scaling](../trade-offs/constraints/vertical-vs-horizontal-scaling.md) · [cache](../trade-offs/architecture/caching-patterns.md) |
| Latency SLO | Caching, async | [latency](../trade-offs/constraints/latency-vs-throughput.md) |
| Consistency needs | CAP | [CAP](../trade-offs/architecture/cap-pacelc-distributed.md) |

---

← [01 — FR](01-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · [03 — API](03-api-design.md) →

Примеры: [instagram-feed.md](../examples/instagram-feed.md) · [paypal-payments.md](../examples/paypal-payments.md) · [vk-social.md](../examples/vk-social.md)

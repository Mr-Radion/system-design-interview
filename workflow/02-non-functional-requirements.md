# Шаг 2 — NFR + trade-offs

← [FRAMEWORK](../FRAMEWORK.md)

## Что фиксируем

Цифры с собеса + NFR **по этапам** (2.1 → 2.7). **Без имён технологий** в §2 — только требования, SLO, failure modes. Trade-off ссылки — **одной строкой в конце подсекции**, не в каждой ячейке.

**Infra и деревья решений** — после шага 5 (HLD), см. [example §6](../examples/instagram-feed.md#6-technology-choices).

## Шаблон §2 (подсекции)

### 2.1 Входные допущения

| Параметр | Значение |
|----------|----------|
| … | … |

**Драйвер дизайна:** что из допущений задаёт bottleneck (FR-…).

### 2.2 Capacity

| Метрика | Формула | Результат |
|---------|---------|-----------|
| … | … | **…** |

**Вывод:** … → связь с FR-ID и §6.

### 2.3 CAP / Consistency

| Участок | Требование | Почему |
|---------|------------|--------|
| … | strong / eventual | 1 фраза |

→ [CAP](../trade-offs/architecture/cap-pacelc-distributed.md)

### 2.4 Latency

#### A. Sync — клиент **ждёт**

| Этап | p50 | p99 |
|------|-----|-----|
| … | … | … |
| **Итого** | … | **≤ SLO** |

#### B. Async — клиент **не ждёт**

| Процесс | E2E SLO | FR |
|---------|---------|-----|
| … | … | FR-… |

### 2.5 Throughput

Peak … · burst-сценарий … · headroom ×N (из §2.2).

### 2.6 Availability & Failure modes

| Параметр | Значение |
|----------|----------|
| SLA | … |
| RPO / RTO | … |

| Сбой | Поведение | FR |
|------|-----------|-----|
| … | … | FR-… |

### 2.7 Observability

| Метрика | Зачем | FR / NFR |
|---------|-------|----------|
| `metric_name` | … | FR-… |

### Traceability (FR → NFR → §6)

| FR | NFR driver | Решение в §6 |
|----|------------|--------------|
| FR-… | … | … |

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

## Чеклист + trade-offs

| Блок | Темы | Trade-offs |
|------|------|------------|
| **Performance** | Latency SLA · RPS · Capacity · CDN · cost | [latency](../trade-offs/constraints/latency-vs-throughput.md) · [cost](../trade-offs/constraints/performance-vs-cost.md) · [CDN](../trade-offs/architecture/cdn-object-storage-pattern.md) |
| Scalability | vertical/horizontal · scalability vs perf · autoscaling · cache | [scaling](../trade-offs/constraints/vertical-vs-horizontal-scaling.md) · [scal vs perf](../trade-offs/constraints/scalability-vs-performance.md) · [autoscale](../trade-offs/constraints/autoscaling-vs-fixed-capacity.md) · [cache](../trade-offs/architecture/caching-patterns.md) |
| Consistency | CAP · PACELC · strong/eventual | [CAP](../trade-offs/architecture/cap-pacelc-distributed.md) · [consistency](../trade-offs/constraints/consistency-as-nfr.md) |
| Reliability | SLA · RPO/RTO · DR · circuit breaker · backpressure | [SLO/RPO](../trade-offs/constraints/availability-slo-rpo-rto.md) · [DR](../trade-offs/architecture/disaster-recovery-pattern.md) · [resilience](../trade-offs/architecture/resilience-backpressure.md) |
| Observability | metrics · logs · traces | [observability](../trade-offs/architecture/observability-architecture.md) · [monitoring](../trade-offs/technologies/monitoring-tools.md) |
| Processing | batch vs stream (Cron) | [batch/stream](../trade-offs/architecture/batch-vs-stream.md) |
| Security | auth · rate limit · signed URL | [gateway](../trade-offs/technologies/api-gateways.md) |
| **Infra** | DB · cache · broker · S3 · gateway · LB · monitoring | [meta](../trade-offs/technologies/technology-selection-meta.md) · [DB](../trade-offs/technologies/databases.md) · [cache](../trade-offs/technologies/caches.md) · [broker](../trade-offs/technologies/message-brokers.md) · [S3](../trade-offs/technologies/object-storage.md) · [gateway](../trade-offs/technologies/api-gateways.md) · [LB](../trade-offs/technologies/load-balancers-proxies.md) |

## Формат одной темы

> **Трейдоф** → **A** / **B** → **✅ Выбор**

## Infra — после HLD (шаг 5 → example §6)

Дерево решений на каждый класс (broker, cache, DB…) + итоговая таблица с продуктами и sizing. Шаблон — [instagram-feed §6](../examples/instagram-feed.md#6-technology-choices).

---

← [01 — FR](01-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · [03 — API](03-api-design.md) →

Примеры: [instagram-feed.md](../examples/instagram-feed.md) · [paypal-payments.md](../examples/paypal-payments.md) · [vk-social.md](../examples/vk-social.md)

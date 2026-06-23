---
layer: pattern
steps: [5]
related:
  - technologies/monitoring-tools
  - constraints/availability-slo-rpo-rto
---

# Observability (наблюдаемость: метрики, логи, трейсы)

> **Главное:** Observability — instrumentation & tracing. Вход — need for SLOs, debugging.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need distributed tracing? | Да | Tracing + metrics |
| Need real-time alerting? | Да | Metrics + logs pipeline |

## Цепочка решений

Шаг 2 NFR → шаг 5 observability pattern → шаг 2 Infra tools

## Three Pillars

| Pillar | Question | When critical |
|--------|----------|---------------|
| Metrics | How much/how fast? | SLO (Service Level Objective, цель уровня сервиса) dashboards, alerting |
| Logs | What happened? | Debug, audit |
| Traces | Where time spent? | Latency debugging in microservices |

## Push vs Pull Metrics

### Push (StatsD, Telegraf agent → collector)

- ➕ **Плюсы:** works for ephemeral (Lambda, short containers); no SD needed.
- ➖ **Минусы / Цена:** collector DDoS risk; no server-side backpressure.
- 📍 **Где:** serverless, IoT devices.

### Pull (Prometheus scrapes `/metrics`)

- ➕ **Плюсы:** server controls scrape rate; easy «target down» detection.
- ➖ **Минусы / Цена:** service discovery required; misses short-lived jobs.
- 📍 **Где:** Kubernetes, long-running services.

## Sampling vs Full Tracing

- **Full:** every request traced → expensive at 100K RPS (Requests Per Second, запросов в секунду).
- **Sampled:** 1-10% + always on errors → cost effective.

## Centralized vs Distributed Logging

- Centralized (ELK, Loki): one search; single point to secure.
- Agent per node → ship to central.

## Black-box vs White-box Monitoring

| | Black-box | White-box |
|--|-----------|-----------|
| Method | Synthetic probes | Internal metrics |
| Catches | User-visible outage | Degradation before outage |

см. [Infra-таблицу шага 2](../../workflow/02-non-functional-requirements.md).


---

## Сокращения в этом файле

| Аббревиатура | Расшифровка |
|--------------|-------------|
| FR | Functional Requirements — функциональные требования |
| NFR | Non-Functional Requirements — нефункциональные требования |
| HLD | High-Level Design — высокоуровневый дизайн |
| RPS | Requests Per Second — запросов в секунду |
| CAP | Consistency, Availability, Partition tolerance |
| LB | Load Balancer — балансировщик нагрузки |

Полный индекс: [README.md](../../FRAMEWORK.md#trade-offs).

---
layer: pattern
steps: [5]
related:
  - constraints/latency-vs-throughput
---

# Batch vs Stream (пакетная vs потоковая обработка)

> **Главное:** Batch vs Stream — выбор обработки данных. Вход — latency vs throughput.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужна near-real-time? | Да | Stream |
| Large bulk processing OK? | Да | Batch |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.3 processing → Deep Dive §4.x (tech)

## Batch Processing (Hadoop, Spark, EMR)

**Выбор:** collect data → process in scheduled chunks (hourly/daily).

- ➕ **Плюсы:** max throughput; efficient resource use; deep historical analysis; cheaper infra.
- ➖ **Минусы / Цена:** high latency (hours/days); stale insights; not for real-time decisions.
- 📍 **Где применять:** daily billing, ETL (Extract, Transform) training, monthly reports.

## Stream Processing (Flink, Kafka Streams)

**Выбор:** process each event as it arrives.

- ➕ **Плюсы:** ms latency; fraud detection; live dashboards; immediate alerts.
- ➖ **Минусы / Цена:** complex (watermarks, windowing, late events); higher infra cost; exactly-once hard.
- 📍 **Где применять:** fraud, stock trading, real-time analytics, IoT alerts.

## Micro-batch (Spark Streaming)

- Компромисс: near-real-time (seconds) with batch efficiency.

## Lambda vs Kappa Architecture

| Arch | Batch + Stream | Stream-only replay |
|------|----------------|-------------------|
| Lambda | ✅ dual pipeline | — |
| Kappa | — | ✅ single stream, reprocess |

## Event Time vs Processing Time

- **Event time:** when event happened (correct for analytics).
- **Processing time:** when processor saw it (simpler, wrong under lag).

---

## Примечания (Habr, часть 5) — Big Data

### Build distributed system vs use Spark

| | Custom coordinator/workers | Apache Spark |
|--|---------------------------|--------------|
| Engineering | Months/years | Days (business logic only) |
| Fault recovery | You implement | Built-in |
| When | Unique constraints | Standard batch/analytics |

**Spark model:** Client → Driver (coordinator) → splits data → Executors (workers) → merge → return.

Coordinator responsibilities: worker failover, rebalance partitions, merge results, logging.

### Когда нужен Big Data tool

- Single machine OOM or too slow.
- ML (Machine Learning, машинное обучение) model training, social network analysis, multi-source ETL (Extract, Transform, Load).

**Interview:** знать coordinator/worker pattern; не кодить Spark internals.

Detail: [distributed-coordination.md](distributed-coordination.md)

**Источник:** [часть 5](https://habr.com/ru/articles/900396/)


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

---
layer: pattern
steps: [5]
related:
  - architecture/batch-vs-stream
  - architecture/messaging-patterns
  - data/replication-sync-async
  - constraints/latency-vs-throughput
---

# ETL Pipeline Pattern (паттерн ETL-конвейера)

> **Главное:** ETL — как перенести данные из источника (часто **PostgreSQL OLTP**) в приёмник (DWH, **Elasticsearch**, lake).  
> Вход — **NFR задержки** + объём + допустимость batch.  
> Выход — **пакетный DAG** (Airflow) или **потоковый CDC** (Debezium + Kafka).

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужен дашборд / search **с задержкой секунды**? | Да | Stream CDC |
| Достаточно отчёта **раз в час/день**? | Да | Batch ETL |
| Источник — **PostgreSQL**, нужны только изменения? | Да | Logical replication + Debezium |
| Большой **полный** снимок по расписанию? | Да | Batch extract (Airflow) |
| Transform тяжёлый, stateful? | Да | Flink / Spark между Kafka и sink |
| Малый объём между шагами batch? | Да | Airflow XCom |
| Большой объём между шагами batch? | Да | Staging S3 / temp table |

## Цепочка решений

Шаг 1 FR (analytics UC) → шаг 2 NFR → **этот паттерн** → Deep Dive §4.x Airflow / Kafka Connect

Подробно с примерами PostgreSQL → Elasticsearch: [references/etl-for-postgresql-se.md](../../references/etl-for-postgresql-se.md).

## Batch ETL (пакетный)

**Выбор:** накопить данные → обработать **пакетом** по **расписанию**.

- ➕ **Плюсы:** большие объёмы; предсказуемая нагрузка; повторный запуск DAG; проще debug.
- ➖ **Минусы / Цена:** latency до следующего run; данные могут устареть в пути.
- 📍 **Где:** статистика продаж за неделю/месяц; nightly sync OLTP → search index.

**Паттерн:** Airflow **DAG** — `extract >> transform >> load` (acyclic, one direction).

**Пример (SE):** `PostgreSQL sales` → JSON transform → `Elasticsearch` → Kibana.

## Stream ETL / CDC (потоковый)

**Выбор:** каждое **изменение** в источнике → событие → transform → load **сразу**.

- ➕ **Плюсы:** near-real-time; не ждём batch window.
- ➖ **Минусы / Цена:** сложнее ops; at-least-once + idempotency; нагрузка скачет.
- 📍 **Где:** live-дашборд заказов; IoT; «новый заказ → сразу в search».

**Паттерн (SE, PostgreSQL):**

1. Logical replication + replication slot  
2. **Debezium** → Kafka topics  
3. Sink connector → Elasticsearch (или другой приёмник)

## Batch vs Stream — когда что (SE)

| | Batch | Stream |
|--|-------|--------|
| Trigger | Schedule | Event / CDC |
| Volume | Large chunks OK | Continuous, smaller events |
| Delay | Hours OK | Seconds |
| Debug | Re-run DAG | Harder replay |
| PostgreSQL extract | SQL dump / incremental batch | Debezium CDC |

Верхний уровень «batch vs stream» → [batch-vs-stream](batch-vs-stream.md).

## Lambda / гибрид

Если нужны **и** исторический batch **и** live updates:

- **Batch** — полная/инкрементальная загрузка ночью (Airflow)
- **Stream** — CDC дельта днём (Debezium)
- Согласовать **idempotency** и версии документов в Elasticsearch


---

## Сокращения в этом файле

| Аббревиатура | Расшифровка |
|--------------|-------------|
| ETL | Extract, Transform, Load |
| CDC | Change Data Capture — захват изменений |
| DAG | Directed Acyclic Graph — граф задач Airflow |
| OLTP / OLAP | Transactional / Analytical processing |
| FR / NFR | Functional / Non-Functional Requirements |

Полный индекс: [README.md](../../FRAMEWORK.md#trade-offs).

## Источники

- [Systems.Education — ETL для PostgreSQL (Airflow, Debezium/Kafka)](https://systems.education/etl-for-postgresql) — [выжимка](../../references/etl-for-postgresql-se.md)

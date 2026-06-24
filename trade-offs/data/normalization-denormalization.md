---
layer: pattern
steps: [4]
related:
  - data/indexing-strategy
---

# Normalization vs Denormalization (нормализация vs денормализация)

> **Главное:** Normalization vs Denormalization — trade-off модели для read/write профилей.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Read-heavy with simple queries? | Да | Denormalize |
| Complex transactional updates? | Да | Normalize |

## Цепочка решений

шаг 2 NFR → HLD §3.2 schema → Deep Dive §4.x → Deep Dive §4.x (tech)

## Normalization (3NF)

**Выбор:** данные без дублирования, связи через FK.

- ➕ **Плюсы:** consistency (update in one place); меньше disk; no anomaly.
- ➖ **Минусы / Цена:** JOIN-heavy reads; slow at scale; N+1 queries.
- 📍 **Где применять:** OLTP (Online Transaction Processing, транзакции) core entities, frequently updated data (user account).

## Denormalization

**Выбор:** дублирование связанных данных в одной записи/document.

- ➕ **Плюсы:** single-query reads; fast feed/timeline; no JOIN compute.
- ➖ **Минусы / Цена:** redundancy; update propagation (user renamed → millions docs); storage ↑.
- 📍 **Где применять:** read-heavy feeds, analytics snapshots, embedded comments.

- 📍 **Where:** LAMP stacks, WordPress-scale, HA standby.

## Materialized views

**Gate:** тяжёлая агрегация на каждый read? → MV + periodic refresh (не live JOIN каждый раз).

- ➕ **Плюсы:** precomputed snapshot; быстрый read dashboard/report.
- ➖ **Минусы:** stale до refresh; storage; invalidation logic.
- 📍 **Где:** analytics on OLTP без полного ETL pipeline; rating aggregates.

## Правило

Normalize writes, denormalize reads — частый hybrid (CQRS (Command Query Responsibility Segregation, разделение команд и запросов)-lite).

**Пример Twitter feed:** denormalized `timeline` table with tweet text + author name embedded; update author name → async fan-out job.


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

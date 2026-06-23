---
layer: paradigm
steps: [4]
related:
  - technologies/databases
  - data/normalization-denormalization
---

# SQL vs NoSQL (реляционные vs нереляционные БД)

> **Главное:** SQL vs NoSQL — парадигмы моделирования данных. Вход — требования к консистентности и запросам (шаг 2).

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужны сложные joins & ACID? | Да | SQL |
| Нужна шардируемость и write scale? | Да | NoSQL |

## Цепочка решений

Шаг 2 NFR → шаг 4 data model → шаг 5 паттерн → шаг 2 Infra tech

## SQL (Relational)

**Выбор:** табличная модель, ACID (Atomicity, Consistency, Isolation, Durability), JOIN, strict schema.

- ➕ **Плюсы:** ACID (Atomicity, Consistency, Isolation, Durability) transactions, complex JOINs, data integrity, mature tooling, strong consistency.
- ➖ **Минусы / Цена:** horizontal scale сложен (sharding manual); vertical scale limit; schema migrations.
- 📍 **Где применять:** banking, e-commerce orders, ERP, anything with complex relations.

## NoSQL — подтипы

### Document (MongoDB-like)
- ➕ Flexible schema, nested JSON (JavaScript Object Notation), fast dev.
- ➖ JOIN weak; consistency varies.
- 📍 User profiles, CMS, catalogs.

### Key-Value (Redis/DynamoDB-like)
- ➕ O(1) access, extreme throughput.
- ➖ No complex queries.
- 📍 Sessions, caching, feature flags.

### Wide-Column (Cassandra-like)
- ➕ Write-heavy, horizontal scale, tunable consistency.
- ➖ Query by partition key only.
- 📍 Time-series, IoT, messaging metadata.

### Graph (Neo4j-like)
- ➕ Traversals, recommendations, social graph.
- ➖ Not for bulk analytics.
- 📍 LinkedIn connections, fraud rings.

## Когда что (GfG)

| Need | Choice |
|------|--------|
| ACID (Atomicity, Consistency, Isolation, Durability) + relations | SQL |
| Schema flexibility + scale | Document NoSQL |
| Extreme write throughput | Wide-column |
| Graph traversals | Graph DB |

**Шаг 4:** выбираем парадигму. см. [Infra-таблицу шага 2](../../workflow/02-non-functional-requirements.md).


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

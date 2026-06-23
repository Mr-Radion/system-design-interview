---
layer: product
steps: [6]
related:
  - data/sql-vs-nosql-paradigm
  - architecture/cap-pacelc-distributed
---

# Databases (выбор СУБД)

> **Главное:** Databases — selection of DB products. Вход — data model, consistency, scale.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need strong transactions? | Да | Relational DB |
| Need massive write scale? | Да | Wide-column / distributed KV |

## Цепочка решений

Шаг 5 pattern → шаг 2 Infra product

## Relational (реляционные СУБД)

### PostgreSQL
- ➕ JSON (JavaScript Object Notation) support, extensions, ACID (Atomicity, Consistency, Isolation, Durability), open source, strong ecosystem.
- ➖ Single-node write scale limit (before sharding).
- 📍 **Default choice** for most relational needs.

### MySQL / MariaDB
- ➕ Wide hosting support, read replicas mature, simpler ops for some.
- ➖ Weaker JSON (JavaScript Object Notation)/analytics vs PG; Oracle licensing (MySQL enterprise).
- 📍 **Where:** LAMP stacks, WordPress-scale, read-heavy replicas.

### CockroachDB / Google Spanner
- ➕ Distributed SQL, horizontal scale, strong consistency globally.
- ➖ Higher latency on writes; cost; complexity.
- 📍 **Where:** global strong consistency required, outgrow single PG.

## Document (документные БД)

### MongoDB
- ➕ Flexible schema, good dev UX (User Experience, пользовательский опыт), horizontal sharding built-in.
- ➖ Memory usage; consistency tuning needed; JOIN limited.
- 📍 **Where:** catalogs, CMS, rapid prototyping document model.

### DynamoDB
- ➕ Serverless scale, predictable latency at AWS scale, pay per request.
- ➖ Vendor lock-in; query patterns must be designed upfront; no ad-hoc JOIN.
- 📍 **Where:** AWS-native, key-value access patterns known.

## Wide-Column (ширококолоночные)

### Cassandra / ScyllaDB
- ➕ Write-heavy, linear scale, multi-region AP.
- ➖ Query by partition key only; ops complexity.
- 📍 **Where:** IoT, messaging, time-series at massive scale.

## When to pick what

| Requirement | Pick |
|-------------|------|
| ACID (Atomicity, Consistency, Isolation, Durability) + relations | PostgreSQL |
| AWS serverless + known access patterns | DynamoDB |
| Write-heavy, global scale | Cassandra |
| Global strong SQL | CockroachDB/Spanner |


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

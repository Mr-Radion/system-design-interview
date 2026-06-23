---
layer: pattern
steps: [4]
---

# Indexing Strategy (стратегия индексирования)

> **Главное:** Indexing — повышает read performance, ухудшает write. Вход — query patterns (шаг 2).

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Частые фильтры/сортировки? | Да | Indexes |
| High write throughput? | Да | Sparse / partial indexes |

## Цепочка решений

Шаг 2 queries → шаг 4 schema/index → шаг 2 Infra tech

## С индексами (B-Tree, Hash, GIN, Composite)

- ➕ **Плюсы:** SELECT/WHERE/ORDER BY на порядки быстрее; covering index → index-only scan.
- ➖ **Минусы / Цена:** INSERT/UPDATE/DELETE медленнее (rebuild tree); RAM (Random Access Memory, оперативная память) + disk; wrong index = wasted space.
- 📍 **Где применять:** columns в WHERE, JOIN, ORDER BY с high selectivity.

## Без индексов (или minimal)

- ➕ **Плюсы:** fast writes; less storage; simpler maintenance.
- ➖ **Минусы / Цена:** full table scan; unusable at millions of rows.
- 📍 **Где применять:** append-only logs, small tables, write-heavy counters (better: separate strategy).

## Composite vs Single-column

| Type | Когда |
|------|-------|
| Single | One filter column dominant |
| Composite (a,b,c) | Multi-column WHERE; order matters (leftmost prefix) |
| Covering | Index contains all SELECT columns → no table lookup |

## Partial Index

- Index only `WHERE status = 'active'` → smaller, faster for hot subset.

**Правило:** index for read patterns from step 1 FR (Functional Requirements, функциональные требования), not «index everything».

---

## Примечания (Habr, часть 2)

### Как работает индекс

- **Без индекса:** full table scan — O(N), каждая строка проверяется.
- **С B-Tree индексом:** O(log N) — id отсортированы, binary search по дереву.
- Primary Key в PostgreSQL → индекс создаётся автоматически.

```sql
CREATE INDEX idx_users_email ON users(email);
-- Одна строка — B-tree строит БД сама
```

### Когда индекс + partition

Большой индекс на огромной таблице → медленный search. **Partitioning** делит таблицу → у каждой partition свой меньший индекс → быстрее.

Query `WHERE created_at BETWEEN '2023-02-01' AND '2023-03-01'` → PostgreSQL идёт только в partition `users_2023_02` (partition pruning).

**Источник:** [часть 2](https://habr.com/ru/articles/877312/)


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

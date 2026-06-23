# Шаг 4 — Data Model

← [FRAMEWORK](../FRAMEWORK.md)

## Схема

**ER (Mermaid `erDiagram`)** в начале §4 — entities + PK/FK. Затем схемы: store split · sharding · replication. Примеры: [instagram](../examples/instagram-feed.md#4-data) · [paypal](../examples/paypal-payments.md#4-data) · [vk-social](../examples/vk-social.md#4-data)

| Store | Ключ | Индекс |
|-------|------|--------|
| … | PK | под запрос |

**Парадигма:** SQL / NoSQL / cache — одна строка «почему».

**Consistency:** strong / eventual — по операции.

## Trade-offs

| Тема | Файл |
|------|------|
| SQL / NoSQL | [sql-nosql](../trade-offs/data/sql-vs-nosql-paradigm.md) |
| Norm / denorm | [norm-denorm](../trade-offs/data/normalization-denormalization.md) |
| Indexing | [indexing](../trade-offs/data/indexing-strategy.md) |
| Sharding / partition | [sharding](../trade-offs/data/sharding-partitioning.md) |
| Replication | [replication](../trade-offs/data/replication-sync-async.md) |
| Master / multi-master | [master-slave](../trade-offs/data/master-slave-multi-master.md) |

---

← [03 — API](03-api-design.md) · [FRAMEWORK](../FRAMEWORK.md) · [05 — Arch](05-architectural-characteristics.md) →

Пример: [instagram-feed.md](../examples/instagram-feed.md)

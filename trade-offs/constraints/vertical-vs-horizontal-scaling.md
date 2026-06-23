---
layer: constraint
steps: [2, 5]
related:
  - architecture/stateless-stateful
  - architecture/caching-patterns
  - data/replication-sync-async
---

# Vertical vs Horizontal Scaling (вертикальное vs горизонтальное масштабирование)

> **Главное:** Vertical — быстро; horizontal — когда упёрлись в железо. Вход — current load. Выход — scaling ladder.

## Что определяет выбор

| Сигнал | Выбор |
|--------|-------|
| Нужен boost за день | Vertical |
| CPU/RAM maxed | Horizontal |
| Read-heavy | Vertical + read replicas |

## Цепочка решений

Vertical → indexes → partition → replicas → sharding ([sharding](../data/sharding-partitioning.md))

## Vertical Scaling (scale-up) vs Horizontal Scaling

### Vertical Scaling

**Выбор:** увеличить CPU (Central Processing Unit, процессор)/RAM (Random Access Memory, оперативная память)/SSD одной машины.

- ➕ **Плюсы:** быстрый выигрыш, простота, нет distributed complexity, ACID (Atomicity, Consistency, Isolation, Durability) проще.
- ➖ **Минусы / Цена:** потолок железа, single point of failure, дорогие large instances.
- 📍 **Где применять:** ранняя стадия, DB до ~32 vCPU, quick win перед реархитектурой.

### Horizontal Scaling

**Выбор:** добавить больше машин + load balancer.

- ➕ **Плюсы:** почти бесконечный scale, fault tolerance, scale отдельных компонентов.
- ➖ **Минусы / Цена:** stateless requirement, distributed transactions, data sharding, DevOps overhead.
- 📍 **Где применять:** web tier, read replicas, stateless API (Application Programming Interface, программный интерфейс), Cassandra/DynamoDB.

## Пример Booking.com

1. **Vertical:** DBMS — больше CPU (Central Processing Unit, процессор), RAM (Random Access Memory, оперативная память), faster disks (immediate boost).
2. **Vertical limit reached → Horizontal:** read replicas + sharding by region.
3. **Trade-off:** complexity ↑, но load growth продолжается.

## Чеклист (Checklist) «10x load» (NFR (Non-Functional Requirements, нефункциональные требования) → HLD (High-Level Design, высокоуровневый дизайн)

1. Stateless? → если нет, вынести state в DB/Cache
2. Horizontal scaling через LB (Load Balancer, балансировщик нагрузки)
3. Cache hot reads
4. Heavy ops → async queue + workers
5. Auto-scaling

**Medium:** «Start vertical. Prepare for horizontal.»

---

## Примечания (Habr, часть 1)

### Vertical — когда предпочитают

- SQL databases (single writer simplicity).
- **Stateful applications** — horizontal scale усложняет session consistency.

### Horizontal — обязательно с LB (Load Balancer, балансировщик нагрузки)

Клиенты не могут выбирать из списка IP. Один domain → **Load Balancer** → least loaded instance (EC2 example: 3 instances, 3 clients, even distribution).

### Auto-scaling

См. [autoscaling-vs-fixed-capacity.md](autoscaling-vs-fixed-capacity.md).

**Источник:** [часть 1](https://habr.com/ru/articles/873388/)

---


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

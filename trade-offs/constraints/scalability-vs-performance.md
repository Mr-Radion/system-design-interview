---
layer: constraint
steps: [2]
related:
  - constraints/vertical-vs-horizontal-scaling
  - architecture/monolith-microservices
---

# Scalability vs Performance (Масштабируемость vs Производительность)

> **Главное:** Когда **растёт нагрузка** — сначала performance (дешевле), потом scalability. Вход — curve роста RPS. Выход — vertical vs horizontal.

## Что определяет выбор

| Сигнал | Действие |
|--------|----------|
| RPS растёт, CPU < 70% | Performance: indexes, cache, code |
| Vertical limit близко | Scalability: stateless + horizontal |
| Непредсказуемые пики | Auto-scaling |

## Цепочка решений

Шаг 2 RPS growth → performance tweaks → horizontal + LB → шаг 2 Infra

## Scalability (масштабируемость) vs Performance (производительность)

**Scalability** — способность системы расти под нагрузкой (больше users, data, RPS (Requests Per Second, запросов в секунду).  
**Performance** — насколько быстро система выполняет *одну* операцию на текущем железе.

### Оптимизация Performance

- ➕ **Плюсы:** быстрый отклик без изменения архитектуры; tuning индексов, CPU (Central Processing Unit, процессор), in-memory.
- ➖ **Минусы / Цена:** упираетесь в потолок одной машины; оптимизация одного hot path не спасает при 10x росте users.
- 📍 **Где применять:** MVP, до product-market fit, когда DAU (Daily Active Users, ежедневные активные пользователи) < 100K.

### Оптимизация Scalability

- ➕ **Плюсы:** система переживает 10x–100x рост; horizontal scale-out, sharding, async.
- ➖ **Минусы / Цена:** распределение нагрузки добавляет overhead (сеть, coordination) → может **снизить** performance одного запроса.
- 📍 **Где применять:** Twitter feed, Instagram stories, SaaS с предсказуемым ростом.

### Ключевой trade-off

Нельзя максимизировать оба одновременно: распределение workloads (scale-out) вводит latency и complexity, снижая per-request speed.

**На шаге 2:** определите, что важнее для текущей фазы — «быстро сейчас» или «выдержать рост через 12 месяцев».


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

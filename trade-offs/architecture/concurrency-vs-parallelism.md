---
layer: pattern
steps: [5]
---

# Concurrency vs Parallelism (конкурентность vs параллелизм)

> **Главное:** Concurrency vs Parallelism — модель выполнения. Вход — task granularity и CPU модель.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Много IO-bound tasks? | Да | Concurrency (async) |
| CPU-bound heavy tasks? | Да | Parallel (multi-thread/process) |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.x execution → Deep Dive §4.x (tech)

## Concurrency (конкурентность)

**Выбор:** одно ядро переключается между задачами (goroutines, async I/O).

- ➕ **Плюсы:** handle many I/O-bound tasks without blocking; upload + thumbnail gen «одновременно» on one core.
- ➖ **Минусы / Цена:** не ускоряет CPU (Central Processing Unit, процессор)-bound single task; scheduling overhead.
- 📍 **Где применять:** web servers, API (Application Programming Interface, программный интерфейс) handlers, I/O wait heavy workloads.

## Parallelism (параллелизм)

**Выбор:** несколько ядер реально выполняют work simultaneously.

- ➕ **Плюсы:** speed up CPU (Central Processing Unit, процессор)-bound (video encode, image resize, ML (Machine Learning, машинное обучение) inference).
- ➖ **Минусы / Цена:** thread sync, memory overhead; diminishing returns (Amdahl's law).
- 📍 **Где применять:** thumbnail generator processing frames on 8 threads.

## Как выбрать thread model

| Workload | Model |
|----------|-------|
| I/O-bound (DB, HTTP (HyperText Transfer Protocol) | Concurrent (async/goroutines) |
| CPU (Central Processing Unit, процессор)-bound (encode) | Parallel (thread pool = CPU (Central Processing Unit, процессор) cores) |
| Mixed | Pipeline: concurrent orchestration + parallel workers |

**Пример:** upload service = concurrent; frame processor inside = parallel.


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

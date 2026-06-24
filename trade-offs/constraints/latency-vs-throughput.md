---
layer: constraint
steps: [2]
related:
  - architecture/caching-patterns
  - architecture/batch-vs-stream
---

# Latency vs Throughput (Задержка vs Пропускная способность)

> **Главное:** Latency/Throughput — **не технология**, а **цифры шаг 2 NFR**. Вход — тип UX и объём. Выход — приоритет для паттернов Deep Dive §4.x.

## Что определяет выбор

| Вопрос | Если «да» | Приоритет |
|--------|-----------|----------|
| Пользователь ждёт ответ «сейчас»? | Real-time UX | **Latency** |
| Миллионы событий/час, задержка OK? | Analytics, billing | **Throughput** |
| И то и другое? | Задайте оба числа: p99 **и** peak RPS | **Balance** |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.x: cache/CDN (latency) или batch/queue (throughput) → Deep Dive §4.x: Redis vs Spark

## Latency (задержка) vs Throughput (пропускная способность)

**Latency** — время от запроса до ответа (ms). **Throughput** — сколько операций система обрабатывает за единицу времени (RPS (Requests Per Second, запросов в секунду), TPS (Transactions Per Second, транзакций в секунду).

### Low Latency, Lower Throughput

- ➕ **Плюсы:** мгновенный UX (User Experience, пользовательский опыт), критично для real-time (трейдинг, гейминг, поиск водителя).
- ➖ **Минусы / Цена:** ресурсы на быстрый ответ одному клиенту; сложнее обрабатывать массовые batch-операции параллельно.
- 📍 **Где применять:** Uber match, банковский перевод, autocomplete (< 100ms p99 (99-й перцентиль задержки).

### High Throughput, Higher Latency

- ➕ **Плюсы:** миллионы событий/сек (логи, Transform).
- ➖ **Минусы / Цена:** клиент ждёт дольше; queueing delay при пиках.
- 📍 **Где применять:** nightly billing (кредитные карты), analytics pipeline, log ingestion.

### Как зафиксировать на шаге 2

| Метрика | Пример цели |
|---------|-------------|
| p50 (50-й перцентиль, медиана) latency | 20 ms |
| p99 (99-й перцентиль задержки) latency | 200 ms |
| Peak throughput | 50K RPS (Requests Per Second, запросов в секунду) read |

Оба параметра задаются **вместе**: «50K RPS (Requests Per Second, запросов в секунду) при p99 (99-й перцентиль задержки) ≤ 200ms» — не «быстро И много» без цифр.

---

## Примечания (Habr, часть 1)

### RTT (Round Trip Time)

**RTT (Round Trip Time, время кругового пути)** — время туда-обратно client → server → client. На interview часто используют как синоним latency для одного request.

### Аналогия из статьи

- **Latency** = время одной машины проехать участок (10 min).
- **Throughput** = сколько машин проехало за час (1000 cars/hour).

Можно иметь **низкую latency** но **низкий throughput** (узкая дорога, быстрые машины) и наоборот.

### Twitter back-of-envelope (полный пример)

См. [workflow/02-non-functional-requirements.md](../../workflow/02-non-functional-requirements.md) — блоки Load, Storage, Resources.

**Источник:** [часть 1](https://habr.com/ru/articles/873388/)


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

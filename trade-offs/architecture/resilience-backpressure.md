---
layer: pattern
steps: [5]
related:
  - architecture/caching-patterns
  - architecture/load-balancing-l4-l7
---

# Resilience & Backpressure (отказоустойчивость и противодавление)

> **Главное:** Resilience & Backpressure — защита от overload. Вход — failure modes and traffic patterns.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need graceful degradation? | Да | Circuit breaker / fallback |
| Need flow control? | Да | Backpressure |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.1 resilience → Deep Dive §4.x (tech)

## Rate Limiting (ограничение частоты запросов)

**Выбор:** ограничить RPS (Requests Per Second, запросов в секунду) per user/IP/API (Application Programming Interface, программный интерфейс) key.

- ➕ **Плюсы:** protect downstream; fair usage; DDoS mitigation.
- ➖ **Минусы / Цена:** legitimate users throttled; 429 errors; config tuning.
- 📍 **Algorithms:** Token Bucket (burst OK), Leaky Bucket, Sliding Window (accurate).

## Circuit Breaker (автоматический выключатель)

**Выбор:** при N failures → stop calling downstream (open circuit).

- ➕ **Плюсы:** fail fast; prevent cascade; auto-recovery (half-open probe).
- ➖ **Минусы / Цена:** false opens; degraded functionality during open state.
- 📍 **Где применять:** external API (Application Programming Interface, программный интерфейс) calls, slow ML (Machine Learning, машинное обучение) service, payment gateway.

## Backpressure (противодавление)

**Выбор:** система сигнализирует producer замедлиться при перегрузке.

- ➕ **Плюсы:** stability under spike; no OOM; controlled degradation.
- ➖ **Минусы / Цена:** dropped/slowed requests; complex flow control.
- 📍 **Flow:** limit incoming → queue max length → drop/degrade → throttle → circuit breaker.

## Graceful Degradation (плавная деградация)

**Выбор:** вернуть approximate/stale data вместо error.

- ➕ **Плюсы:** UX (User Experience, пользовательский опыт) preserved; core function alive.
- ➖ **Минусы / Цена:** accuracy ↓ (cached ETA vs real-time).
- 📍 **Пример X-Taxi:** rain spike → cached approximate ETA.

## Bulkhead Pattern

- Isolate thread pools / connection pools per dependency → one slow service doesn't exhaust all resources.

## Batching vs Per-request

| | Batching | Per-request |
|--|----------|-------------|
| Throughput | ↑ | ↓ |
| Latency | ↑ (wait for batch) | ↓ |
| Use | Logs, metrics, bulk API (Application Programming Interface, программный интерфейс) | Interactive UX (User Experience, пользовательский опыт) |

**Пример Spotify:** cached track list + rate limit + queue for ML (Machine Learning, машинное обучение) recommendations.


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

---
layer: pattern
steps: [5]
related:
  - technologies/load-balancers-proxies
  - architecture/stateless-stateful
---

# Load Balancing L4/L7 (балансировка: транспортный vs прикладной уровень)

> **Главное:** Load balancing L4 vs L7 — где балансировать трафик. Вход — routing needs and health checks.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need HTTP-aware routing? | Да | L7 |
| Need TCP passthrough and speed? | Да | L4 |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.1 → Deep Dive §4.x (tech)

## L4 (Transport: TCP (Transmission Control Protocol)/UDP (User Datagram Protocol)

**Примеры:** AWS NLB, HAProxy TCP (Transmission Control Protocol) mode.

- ➕ **Плюсы:** millions RPS (Requests Per Second, запросов в секунду); no HTTP (HyperText Transfer Protocol) parsing; low CPU (Central Processing Unit, процессор); protocol-agnostic.
- ➖ **Минусы / Цена:** no path-based routing; no cookie stickiness; no SSL termination (usually).
- 📍 **Где применять:** DB proxy, gaming UDP (User Datagram Protocol), raw TCP (Transmission Control Protocol) microservices, extreme throughput.

## L7 (Application: HTTP (HyperText Transfer Protocol)/HTTPS (HTTP Secure, HTTP + TLS)

**Примеры:** Nginx, AWS ALB, Envoy.

- ➕ **Плюсы:** path routing (`/api/video` → video cluster); canary/A/B; sticky sessions; SSL termination.
- ➖ **Минусы / Цена:** lower throughput; CPU (Central Processing Unit, процессор) for TLS + parsing; added latency ~1-5ms.
- 📍 **Где применять:** web APIs, canary deploys, multi-tenant routing.

## LB (Load Balancer, балансировщик нагрузки) Algorithms

| Algorithm | Best for |
|-----------|----------|
| Round Robin | Equal capacity servers |
| Weighted RR | Mixed instance sizes |
| Least Connections | Long-lived connections |
| IP Hash | Session affinity without cookies |
| Least Response Time | Heterogeneous latency |

## Power of two choices

LB выбирает **2 random** backend, отправляет на менее loaded. O(1) vs scan all servers. → [GLOSSARY](../../GLOSSARY.md#power-of-two-choices)

## GeoDNS

DNS возвращает IP ближайшего региона (latency routing). Для global users; не заменяет health checks.

## Forward vs Reverse Proxy

| | Forward | Reverse |
|--|---------|---------|
| Client knows? | Yes (configured) | No |
| Use | Corporate egress | Protect backend |

## Sticky Sessions vs Stateless

- Sticky: needed for stateful WebSocket/state in memory.
- Stateless: preferred; externalize session to Redis.

см. [Infra-таблицу шаг 2 NFR](../../workflow/02-non-functional-requirements.md).

---

## Примечания (Habr, часть 3)

### Зачем LB (Load Balancer, балансировщик нагрузки)

Клиент не может держать список IP всех backend servers. Один DNS (Domain Name System, система доменных имён) → LB (Load Balancer, балансировщик нагрузки) → least loaded server.

Связь с horizontal scaling (часть 1): 3 EC2 instances, clients hit LB (Load Balancer, балансировщик нагрузки) domain, not individual IPs.

### Алгоритмы — детали из статьи

**Round Robin:** S1 → S2 → S3 → S1… Simple, но **ignores current load**.

**Weighted RR:** мощный server weight=2 получает 2× requests — static weights, не realtime perf.

**Least Connections:** → server с минимумом active connections (HTTP (HyperText Transfer Protocol), TCP (Transmission Control Protocol), WS). Dynamic, но плохо если connection durations vary wildly.

**Hash-based:** hash(client IP / user_id) → same server always → **session affinity**. Minus: add/remove server breaks hash ring (→ consistent hashing).

**Default в cloud LB (Load Balancer, балансировщик нагрузки):** часто Round Robin или Least Connections — уточняйте у провайдера (AWS ALB ≈ L7 + health checks).

**Источник:** [часть 3](https://habr.com/ru/articles/885054/)


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

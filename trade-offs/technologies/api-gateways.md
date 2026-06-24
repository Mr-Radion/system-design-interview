---
layer: product
steps: [6]
related:
  - api/rest-grpc-graphql
  - architecture/load-balancing-l4-l7
---

# API Gateways (шлюзы API, Application Programming Interface)

> **Главное:** API gateways — exposure, auth, routing. Вход — security and orchestration needs.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need auth + routing at edge? | Да | API Gateway |
| Need low-latency pass-through? | Да | lightweight proxy |

## Цепочка решений

Deep Dive §4.x pattern → Deep Dive §4.x (tech)

## Kong

- ➕ Plugin ecosystem (auth, rate limit, logging); OSS + enterprise; declarative config.
- ➖ Lua plugins learning curve; DB dependency for some modes.
- 📍 **Where:** multi-service API (Application Programming Interface, программный интерфейс) management, plugin-rich needs.

## AWS API (Application Programming Interface, программный интерфейс) Gateway

- ➕ Fully managed; Lambda integration; usage plans; no servers.
- ➖ Cold start with Lambda; vendor lock-in; cost at high RPS (Requests Per Second, запросов в секунду); 29s timeout.
- 📍 **Where:** serverless AWS, external public API (Application Programming Interface, программный интерфейс) with usage tiers.

## Envoy (as Gateway)

- ➕ Same stack as service mesh; gRPC; dynamic routing; observability.
- ➖ Not a «product» out of box; needs Gloo/Contour wrapper for K8s (Kubernetes, оркестратор контейнеров) ingress.
- 📍 **Where:** K8s (Kubernetes, оркестратор контейнеров) + Istio/Contour, gRPC-first.

## Nginx (as API (Application Programming Interface, программный интерфейс) Gateway lite)

- ➕ Simple reverse proxy + rate limit + SSL; team knows it.
- ➖ No built-in API (Application Programming Interface, программный интерфейс) key management, developer portal, analytics.
- 📍 **Where:** internal APIs, simple routing, budget constrained.

## Build vs Buy Gateway

| | Custom (Nginx + code) | Dedicated Gateway |
|--|----------------------|-------------------|
| Features | You build | Auth, quotas, analytics built-in |
| Cost | Dev time | License/hosting |
| Use | 2-3 services | 20+ services, external developers |

## API (Application Programming Interface, программный интерфейс) Gateway vs BFF (Backend-for-Frontend, бэкенд для фронтенда)

- **Gateway:** cross-cutting (auth, rate limit, SSL) for all services.
- **BFF (Backend-for-Frontend, бэкенд для фронтенда):** per-client aggregation (mobile BFF ≠ web BFF); sits behind or alongside gateway.

---

## Примечания (Habr, часть 3)

В серии API (Application Programming Interface, программный интерфейс) Gateway описан как **pattern** (Deep Dive §4.x), не конкретный продукт:

| Function | Why at gateway |
|----------|----------------|
| Single entry point | Hide N microservice IPs |
| Routing | `/users` → user service |
| Rate limiting | Protect backends |
| Caching | Hot public endpoints |
| AuthN/AuthZ | Central security |

На шаге 6 выбираете Kong / AWS API (Application Programming Interface, программный интерфейс) GW / Nginx based on [technology-selection-meta.md](technology-selection-meta.md).

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

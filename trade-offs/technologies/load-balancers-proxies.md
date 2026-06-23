---
layer: product
steps: [6]
related:
  - architecture/load-balancing-l4-l7
---

# Load Balancers & Proxies (балансировщики и прокси)

> **Главное:** Load balancers & proxies — products for L4/L7. Вход — routing, TLS, observability needs.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need L7 routing & TLS termination? | Да | Envoy / Nginx |
| Need simple L4 balancing? | Да | IPVS / ELB |

## Цепочка решений

Шаг 5 pattern → шаг 2 Infra product

## Nginx

- ➕ Mature L7; reverse proxy; SSL termination; static files; low resource; wide knowledge.
- ➖ Config reload; limited dynamic service discovery vs Envoy.
- 📍 **Where:** web APIs, static content, simple L7 routing.

## HAProxy

- ➕ Extreme L4/L7 performance; advanced health checks; TCP (Transmission Control Protocol) expertise.
- ➖ Less «full platform» than Envoy for service mesh.
- 📍 **Where:** TCP (Transmission Control Protocol) passthrough, high RPS (Requests Per Second, запросов в секунду) L4, MySQL/Postgres proxy.

## Envoy

- ➕ Dynamic config (xDS); observability built-in; gRPC native; service mesh sidecar.
- ➖ Complexity; resource heavier than Nginx.
- 📍 **Where:** Kubernetes, Istio/Linkerd, gRPC microservices.

## AWS ALB / NLB

- ➕ Managed; auto-scale; integrated with AWS; NLB = L4 millions RPS (Requests Per Second, запросов в секунду).
- ➖ Vendor lock-in; cost at scale; less flexible than self-hosted.
- 📍 **Where:** AWS deployments, managed ops priority.

## Hardware LB (F5)

- ➕ Extreme throughput; dedicated ASIC; enterprise support.
- ➖ $$$$$; slow to change; overkill for cloud-native.
- 📍 **Where:** telco, legacy enterprise DC.

## Comparison (сравнение)

| Scenario | Pick |
|----------|------|
| Simple web API (Application Programming Interface, программный интерфейс) on VM | Nginx |
| AWS managed | ALB (L7) / NLB (L4) |
| K8s (Kubernetes, оркестратор контейнеров) service mesh | Envoy |
| Raw TCP (Transmission Control Protocol) scale | HAProxy / NLB |


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

---
layer: paradigm
steps: [3]
related:
  - technologies/api-gateways
---

# REST vs gRPC vs GraphQL

> **Главное:** API layer — входной контракт для клиента и сервисов. Вход — фаза 1 FR + шаг 2 NFR (latency/compatibility). Выход — контракт HLD §3.1 и выбор между sync/async и versioning.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Клиент ждёт ответ мгновенно? | Да | Sync (gRPC/REST) |
| Нужна гибкость в эволюции схемы? | Да | GraphQL или versioned REST |
| Интерсёрвисная высокая пропускная способность? | Да | gRPC / binary |

## Цепочка решений

шаг 2 NFR + FR → HLD §3.1 API (контракт) → Deep Dive §4.x паттерн → Deep Dive §4.x технология

## REST (JSON (JavaScript Object Notation) over HTTP (HyperText Transfer Protocol)

- ➕ **Плюсы:** читаемость, универсальная поддержка, HTTP (HyperText Transfer Protocol) caching (CDN (Content Delivery Network, сеть доставки контента), Nginx), огромная экосистема, простое тестирование (curl).
- ➖ **Минусы / Цена:** большой payload (JSON (JavaScript Object Notation) text), over-fetching/under-fetching, нет strict schema из коробки, медленная сериализация.
- 📍 **Где применять:** public API (Application Programming Interface, программный интерфейс), B2B integrations, CRUD, browser-first apps.

## gRPC (Protobuf over HTTP (HyperText Transfer Protocol)/2)

- ➕ **Плюсы:** бинарный compact payload, strict typing (.proto), codegen, streaming, высокая скорость.
- ➖ **Минусы / Цена:** нечитаем без tools, плохая browser support (нужен gRPC-Web proxy), сложнее debug.
- 📍 **Где применять:** inter-service communication, microservices, mobile↔backend high-throughput.

## GraphQL (Graph Query Language)

- ➕ **Плюсы:** клиент выбирает поля → решает over/under-fetching; один endpoint; идеально для mobile.
- ➖ **Минусы / Цена:** N+1 на DB (нужен DataLoader), нет HTTP (HyperText Transfer Protocol) cache (POST), complexity на backend, query depth attacks.
- 📍 **Где применять:** mobile apps с разными экранами, aggregating multiple sources.

## Сравнительная матрица

| Критерий | REST | gRPC | GraphQL |
|----------|------|------|---------|
| Browser | ✅ | ❌ (proxy) | ✅ |
| Caching | ✅ | ❌ | ❌ |
| Payload size | Large | Small | Variable |
| Schema | OpenAPI optional | .proto strict | GraphQL schema |

**Medium:** REST = full thali, GraphQL = choose your meal. Держите REST для простых flows, GraphQL где flexibility критична.


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

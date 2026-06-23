---
layer: paradigm
steps: [3]
---

# API (Application Programming Interface, программный интерфейс) Versioning (версионирование API (Application Programming Interface, программный интерфейс)

> **Главное:** Versioning & evolution — как эволюционировать API безопасно. Вход — backward/forward compat requirements.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Нужна backward совместимость? | Да | Additive changes, versioning |
| Нужен граф запросов? | Да | GraphQL / gateway |

## Цепочка решений

Шаг 2 NFR + FR → шаг 3 API design → шаг 5 pattern → шаг 2 Infra tooling

## Backward-compatible evolution vs Breaking changes

### Backward-compatible (рекомендуется)

- ➕ **Плюсы:** клиенты не ломаются; gradual migration; меньше support tickets.
- ➖ **Минусы / Цена:** payload растёт; deprecated fields живут годами; dual maintenance.
- 📍 **Практика:** добавлять поля, не удалять; не менять semantics существующих полей.

### Breaking changes

- ➕ **Плюсы:** чистый API (Application Programming Interface, программный интерфейс) без legacy baggage.
- ➖ **Минусы / Цена:** forced client updates; mobile apps в store с old versions.
- 📍 **Когда допустимо:** internal API (Application Programming Interface, программный интерфейс), controlled client base.

## Стратегии versioning

| Стратегия | Пример | Плюсы | Минусы |
|-----------|--------|-------|--------|
| URL path | `/api/v1`, `/api/v2` | Explicit, cacheable | URL proliferation |
| Header | `Accept-Version: v2` | Clean URLs | Hidden, harder to test |
| Content negotiation | `Accept: application/vnd.api.v2+json` | RESTful purist | Complex |

## Decision flow (из interview)

1. Ввести versioning (`v1`, `v2`)
2. Добавлять поля, не удалять старые
3. Breaking change needed? → сначала optional → notify clients → parallel versions
4. Не менять semantics существующих полей

**Пример Twitter API (Application Programming Interface, программный интерфейс):** `/api/v1` + `/api/v2`, новые поля optional, старые semantics неизменны.


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

---
layer: pattern
steps: [5]
related:
  - workflow/01-functional-requirements
  - technologies/technology-selection-meta
---

# Monolith vs Microservices (монолит vs микросервисы)

> **Главное:** Monolith vs Microservices — границы разбиения. Вход — организационная модель и coupling.

## Что определяет выбор

| Сигнал (минус монолита проявился?) | Если «да» | Выбор |
|-------------------------------------|-----------|-------|
| Нужен **независимый scale** модуля (CPU/RAM только для media/report)? | Да | Microservices или extract service |
| **Деплой** одной фичи блокирует весь релиз? | Да | Microservices / modular monolith + CI split |
| **Coupling** — правка модуля ломает соседей? | Да | Modular monolith → extract |
| Новый функционал **всегда через ядро**? | Да | Пересмотр границ / microservices |
| **Несколько команд** мешают друг другу в одном repo? | Да | Microservices (Conway's Law) |
| **Strong ACID** across domains обязателен? | Да | Monolith / modular monolith |

## Цепочка решений

Шаг 2 NFR → шаг 5 pattern (deployment) → шаг 2 Infra tech

## Monolith

- ➕ **Плюсы:** fast time-to-market; in-memory calls (zero network latency); simple deploy на старте; easy ACID (Atomicity, Consistency, Isolation, Durability); easy debugging; один артеfact — проще CI/CD в начале.
- ➖ **Минусы / Цена:**
  1. **Слабое масштабирование** — масштабируется **только целиком**; нельзя отдельно увеличить CPU-heavy модуль (медиа, отчёты) без поднятия всего приложения.
  2. **Медленный деплой** — правка в одном модуле → **пересборка и выкладка всего** монолита; длинный цикл релиза при росте кодовой базы.
  3. **Сильная связанность (coupling)** — модули тянут друг друга; **нельзя менять границу** без риска сломать соседей; растёт «big ball of mud».
  4. **Сложность развития** — новый функционал часто требует **изменений в ядре** (общие модели, shared DB, cross-module imports), а не изолированного добавления.
  5. **Ограничения командной работы** — одна большая команда на один репозиторий: **конфликты в git**, блокировки релизов, Conway's Law не работает (организация ≠ границы кода).
  6. **Единая зона отказа** — падение процесса или deploy-баг бьёт по **всем** доменам сразу.
- 📍 **Где применять:** startup MVP, < 10 engineers, bounded context ещё не ясен; Shopify-scale monolith возможен при **modular monolith** и дисциплине.

## Microservices

- ➕ **Плюсы:** independent deploy/scale; fault isolation; polyglot stacks; team autonomy (Conway's Law).
- ➖ **Минусы / Цена:** DevOps/CI/CD/mesh complexity; network latency; distributed transactions (Saga); observability mandatory.
- 📍 **Где применять:** large org, clear bounded contexts, different scaling per service.

## Modular Monolith (компромисс)

- ➕ **Плюсы:** domain boundaries in code; single deploy; path to extract services later (Strangler Fig).
- ➖ **Минусы / Цена:** discipline required; can degrade to spaghetti without enforcement.
- 📍 **Recommended:** default for most teams before microservices.

## Monolith-first vs Microservices-first

**Medium:** «Don't start with microservices too early.» Start monolith → extract when появляются минусы выше:

| Минус монолита | Сигнал «пора делить» |
|----------------|----------------------|
| Масштабирование | Media/report модуль нужен ×10 CPU, остальное — нет |
| Медленный деплой | Releases slowing down, regression на unrelated modules |
| Связанность | Refactor одного домена тянет пол-репозитория |
| Развитие через ядро | Каждая фича — правки в shared kernel / god service |
| Команды | 3+ команд в одном repo, merge queue, blocked releases |

## Strangler Fig vs Big Bang Rewrite

| Pattern | Risk | When |
|---------|------|------|
| Strangler Fig | Low | Gradual migration, zero downtime |
| Big Bang | High | Legacy unmaintainable, hard deadline |

---

## Примечания (Habr, часть 3)

### E-commerce decomposition example

Monolith: users + products + orders + payments in one app.

Microservices: User Service, Product Service, Order Service, Payment Service — separate deploys, separate IPs (inconvenient for clients).

### Почему microservices (из статьи)

1. **Independent scale** — product service 2 machines, user service 3, payment 1.
2. **Polyglot** — users on Node.js, orders on Go.
3. **Fault isolation** — Order service down ≠ User/Product down.

### Когда microservices (Conway's Law)

«Microservices are defined by org structure» — 3 teams → 3 services, grows with teams.

**Стартап:** 2–3 devs → monolith first; migrate when teams grow.

### API (Application Programming Interface, программный интерфейс) Gateway pattern

Client → **single endpoint** (API (Application Programming Interface, программный интерфейс) Gateway) → routes to correct microservice.

Gateway also provides: **rate limiting, caching, auth** — cross-cutting, not business logic.

Product choice: см. [Infra-таблицу шага 2](../../workflow/02-non-functional-requirements.md)

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

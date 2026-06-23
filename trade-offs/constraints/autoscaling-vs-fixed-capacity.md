---
layer: constraint
steps: [2]
related:
  - constraints/vertical-vs-horizontal-scaling
  - constraints/performance-vs-cost
---

# Auto-scaling vs Fixed Capacity (автомасштабирование vs фиксированная ёмкость)

> **Главное:** Fixed — предсказуемый traffic; autoscaling — spikes. Вход — traffic pattern. Выход — capacity model.

## Что определяет выбор

| Pattern | Выбор |
|---------|-------|
| Flat / predictable | Fixed capacity |
| Spikes / seasonal | Auto-scaling |
| Cloud K8s | HPA default |

## Цепочка решений

Traffic pattern → fixed vs HPA/ASG → шаг 2 Infra

## Auto-scaling (динамическое масштабирование)

**Выбор:** мониторинг метрик (CPU (Central Processing Unit, процессор), RPS (Requests Per Second, запросов в секунду), queue depth) → автоматически добавлять/убирать инстансы.

- ➕ **Плюсы:** платите только за нужную мощность; переживаете пики без ручного вмешательства; cost-efficient в off-peak.
- ➖ **Минусы / Цена:** cold start новых инстансов; lag реакции (scale-up не мгновенный); сложность политик и тестирования порогов.
- 📍 **Где применять:** variable traffic (e-commerce sales, SaaS weekday/weekend), cloud-native (K8s HPA (Kubernetes Horizontal Pod Autoscaler), AWS ASG).

## Fixed Max Capacity (всегда максимум инстансов)

**Выбор:** держать 100 инстансов постоянно, хотя в среднем нужно 10.

- ➕ **Плюсы:** zero scale-up delay; простота ops; гарантированный headroom для любого пика.
- ➖ **Минусы / Цена:** переплата 90% времени; idle resources; Performance vs Cost trade-off в минус.
- 📍 **Где применять:** критичные SLA (Service Level Agreement, соглашение об уровне сервиса) без cold start tolerance, black Friday pre-provision, legacy без autoscale.

## Reactive vs Predictive Auto-scaling

| Type | Trigger | Trade-off |
|------|---------|-----------|
| Reactive | CPU (Central Processing Unit, процессор) > 90% сейчас | Просто, но опаздывает на резкий spike |
| Predictive | ML (Machine Learning, машинное обучение)/schedule по истории | Сложнее, но готовит capacity до пика |

---

## Примечания (Habr, часть 1)

**Сценарий из статьи:** 1 инстанс = 1000 users. В пик 100K users → нужно 100 инстансов. В спад — 10K users → достаточно 10.

- **Плохое решение:** всегда держать 100 инстансов → переплата в периоды низкой нагрузки.
- **Хорошее решение:** autoscaling — мониторить CPU (например порог 90%), при превышении добавлять инстанс и распределять трафик через LB (Load Balancer, балансировщик нагрузки).

Цифры гипотетические — реальный порог находят **нагрузочным тестированием** одного инстанса.

**Связь с workflow:** фиксируйте на шаге 2 — допустим ли cold start (p99 (99-й перцентиль задержки) spike при scale-up) или нужен fixed headroom.

**Источник:** [System Design для начинающих, часть 1](https://habr.com/ru/articles/873388/)


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

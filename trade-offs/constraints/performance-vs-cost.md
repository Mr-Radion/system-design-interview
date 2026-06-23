---
layer: constraint
steps: [2]
related:
  - constraints/vertical-vs-horizontal-scaling
  - technologies/technology-selection-meta
---

# Performance vs Cost (Производительность vs Стоимость)

> **Главное:** Budget — constraint шага 2. Вход — SLA и ceiling $. Выход — right-size infra vs premium.

## Что определяет выбор

| Вопрос | Выбор |
|--------|-------|
| SLA 99.99%+ и revenue critical? | Performance > cost |
| MVP / internal tool? | Cost > performance |
| Over-provisioned? | Right-size + autoscaling |

## Цепочка решений

Шаг 2 budget + SLA → managed vs self-hosted → шаг 2 Infra

## Performance (производительность) vs Cost (стоимость)

**Performance** — скорость, latency, throughput. **Cost** — CAPEX/OPEX: железо, managed services, команда DevOps.

### High Performance, High Cost

- ➕ **Плюсы:** лучший UX (User Experience, пользовательский опыт), headroom для пиков, меньше churn.
- ➖ **Минусы / Цена:** premium instances, multi-region, over-provisioning, managed DB ($$$).
- 📍 **Где применять:** fintech, ad-tech RTB, enterprise SLA (Service Level Agreement, соглашение об уровне сервиса) 99.99%.

### Lower Performance, Lower Cost

- ➕ **Плюсы:** стартап-бюджет, spot instances, single-AZ, batch вместо stream.
- ➖ **Минусы / Цена:** медленнее UX (User Experience, пользовательский опыт), риск downtime при пиках, technical debt при росте.
- 📍 **Где применять:** internal tools, MVP, side projects.

### Как зафиксировать на шаге 2

```
Budget: $5K/month infra
Target: p99 (99-й перцентиль задержки) < 500ms (не 50ms — сознательная экономия)
Availability: 99.9% (не 99.99% — экономия multi-region)
```

**Правило:** каждое «+9» в availability ≈ +3–10x cost. Зафиксируйте budget ceiling до HLD (High-Level Design, высокоуровневый дизайн).


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

---
layer: pattern
steps: [5]
related:
  - architecture/load-balancing-l4-l7
  - architecture/stateless-stateful
---

# Deployment / Release Strategies (стратегии выката)

> **Главное:** как выкатывать новую версию без downtime и с контролируемым риском. Вход — release frequency, tolerance к багам, stateless/stateful.

## Цепочка решений

```
Шаг 5 NFR (availability, release cadence) → stateless? → strategy → L7 LB / K8s §7
```

## Стратегии

| Стратегия | Как работает | Риск | Когда |
|-----------|--------------|------|-------|
| **Rolling** | постепенно заменяем pods/instances | средний | default K8s |
| **Blue-Green** | два env, switch traffic | low rollback | critical services |
| **Canary** | % traffic на new version | lowest blast radius | L7 LB, feature flags |

## Gate

| Вопрос | Если да | Выбор |
|--------|---------|-------|
| Нужен instant rollback? | Да | blue-green |
| Хотим тест на 5% users? | Да | canary |
| Stateless + frequent deploys? | Да | rolling OK |

## Связь с LB

Canary и blue-green требуют **L7** routing → [load-balancing-l4-l7](load-balancing-l4-l7.md)

## Резюме

- Stateful (DB migration) — rolling сложнее; миграции отдельно.
- Canary + metrics — минимальный риск для payments/feed.
- Blue-green — дороже (2× infra), быстрый rollback.

## FAQ (собес)

| Вопрос | Ответ |
|--------|-------|
| Rolling vs recreate? | Rolling — zero downtime if health checks OK. |
| Canary vs A/B test? | Canary — deploy risk. A/B — product experiment. |
| DB schema + blue-green? | Backward-compatible migrations first. |

Полный индекс: [FRAMEWORK.md](../../FRAMEWORK.md#trade-offs).

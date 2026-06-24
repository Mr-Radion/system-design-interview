---
layer: product
steps: [6]
related:
  - data/sql-vs-nosql-paradigm
---

# Meta: выбор технологий (Technology Selection)

> **Главное:** Tech selection meta — questions to pick a product. Вход — constraints & ops readiness.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Ops team ready for self-host? | Да | Self-host |
| Need managed for speed? | Да | Managed service |

## Цепочка решений

Deep Dive §4.x pattern → Deep Dive §4.x (tech) selection

## Managed vs Self-hosted (управляемый vs свой хостинг)

| | Managed (RDS, Confluent Cloud) | Self-hosted |
|--|-------------------------------|-------------|
| Ops burden | Low | High |
| Cost at scale | Higher unit cost | Lower at huge scale |
| Control | Limited tuning | Full |
| Use | Most teams, startup | Special requirements, cost at scale |

## Build vs Buy (строить vs покупать)

| | Build | Buy/SaaS |
|--|-------|----------|
| Time | Months | Days |
| Fit | Perfect | 80% |
| Use | Core differentiator | Commodity (auth, email, payments) |

## Best-fit vs Team Expertise (лучший fit vs экспертиза команды)

- ➕ **Expertise:** faster delivery, fewer bugs, easier hiring.
- ➕ **Best-fit:** better performance at scale, lower long-term cost.
- **Правило:** choose boring technology team knows, unless NFR (Non-Functional Requirements, нефункциональные требования) clearly violated.

## Cloud-native vs Portable (облачный vs переносимый)

| | Cloud-native (DynamoDB, SQS) | Portable (Postgres, RabbitMQ) |
|--|------------------------------|-------------------------------|
| Integration | Seamless with AWS/GCP | Manual wiring |
| Lock-in | High | Low |
| Performance | Optimized for cloud | Good everywhere |

## Open Source vs Commercial (открытый vs коммерческий)

- OSS: no license cost, community, self-support.
- Commercial (Datadog, MongoDB Atlas): support, features, predictable SLA (Service Level Agreement, соглашение об уровне сервиса).

## Decision checklist (чеклист решений, Deep Dive §4.x)

1. Какой **pattern** из Deep Dive §4.x? (Cache-Aside, event stream…)
2. Какие **NFR (Non-Functional Requirements, нефункциональные требования)** из шаг 2 NFR? (p99 (99-й перцентиль задержки), RPO (Recovery Point Objective, допустимая потеря данных), budget)
3. Managed или self-hosted?
4. Team знает технологию?
5. Exit strategy (vendor lock-in acceptable?)


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

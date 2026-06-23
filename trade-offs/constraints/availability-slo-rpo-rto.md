---
layer: constraint
steps: [2]
related:
  - architecture/disaster-recovery-pattern
  - architecture/observability-architecture
---

# Availability, SLO, RPO/RTO (доступность, цели сервиса и восстановления)

> **Главное:** Availability/RPO/RTO — **цифры шага 2**. Вход — downtime cost. Выход — DR-паттерн и replication.

## Что определяет выбор

| Tier | Availability | RPO | RTO |
|------|--------------|-----|-----|
| Critical | 99.99%+ | ~0 | < 5 min |
| Standard | 99.9% | 1 h | 4 h |
| Internal | 99% | 24 h | 24 h |

## Цепочка решений

Шаг 2 SLA → multi-AZ / DR pattern → sync vs async repl → шаг 2 Infra

## Availability (доступность)

**Availability** = uptime / (uptime + downtime). «Три девятки» = 99.9% ≈ 8.7h downtime/year.

| SLA (Service Level Agreement, соглашение об уровне сервиса) | Downtime/year | Когда достаточно |
|-----|---------------|------------------|
| 99.9% | ~8.7 h | B2B SaaS, internal |
| 99.99% | ~52 min | E-commerce, payments |
| 99.999% | ~5 min | Banking core, telco |

- ➕ **Высокий SLA (Service Level Agreement, соглашение об уровне сервиса):** доверие пользователей, enterprise contracts.
- ➖ **Цена:** multi-AZ, multi-region, DR (Disaster Recovery, аварийное восстановление) drills, on-call 24/7, $$.

## SLI (Service Level Indicator, индикатор уровня сервиса) / SLO (Service Level Objective, цель уровня сервиса) / SLA (Service Level Agreement, соглашение об уровне сервиса)

- **SLI (Service Level Indicator, индикатор уровня сервиса)** (indicator): «p99 (99-й перцентиль задержки) latency feed read»
- **SLO (Service Level Objective, цель уровня сервиса)** (objective): «p99 (99-й перцентиль задержки) < 200ms в 99.5% времени за 30 дней»
- **SLA (Service Level Agreement, соглашение об уровне сервиса)** (agreement): контракт с penalties при нарушении SLO (Service Level Objective, цель уровня сервиса)

### Strict SLO (Service Level Objective, цель уровня сервиса) vs Relaxed SLO

- ➕ Strict: предсказуемое качество, раннее обнаружение деградации.
- ➖ Strict: дороже infra, больше false alerts, over-engineering.
- 📍 Relaxed SLO (Service Level Objective, цель уровня сервиса): MVP, non-critical features.

## RPO (Recovery Point Objective, допустимая потеря данных) vs RTO (Recovery Time Objective, допустимое время восстановления)

- **RPO (Recovery Point Objective, допустимая потеря данных)**: сколько **данных** можно потерять (1 min / 1 hour / 1 day).
- **RTO (Recovery Time Objective, допустимое время восстановления)**: сколько **времени** на восстановление (5 min / 4 hours).

| Tier | RPO (Recovery Point Objective, допустимая потеря данных) | RTO (Recovery Time Objective, допустимое время восстановления) | Pattern (шаг 5) |
|------|-----|-----|-----------------|
| Critical | 0 | < 5 min | Sync replication, hot standby |
| Standard | 1 h | 4 h | Async replication, warm standby |
| Archive | 24 h | 24 h | Daily backups, cold standby |

**На шаге 2:** зафиксируйте availability %, RPO (Recovery Point Objective, допустимая потеря данных), RTO (Recovery Time Objective, допустимое время восстановления) **до** выбора DR-архитектуры (Disaster Recovery, аварийное восстановление).


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

---
layer: pattern
steps: [5]
related:
  - constraints/availability-slo-rpo-rto
---

# Disaster Recovery (аварийное восстановление, DR (Disaster Recovery, аварийное восстановление)

> **Главное:** Disaster recovery — DR plans and RTO/RPO. Вход — SLA and business impact.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Critical SLA? | Да | Active-active / warm standby |
| Cost-sensitive? | Да | Cold backups |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.4 DR → Deep Dive §4.x (tech)

## Backup Strategies

| Type | RPO (Recovery Point Objective, допустимая потеря данных) | Storage | Restore speed |
|------|-----|---------|---------------|
| Full | High (24h) | Large | Slow |
| Incremental | Medium | Small | Medium chain |
| Differential | Medium | Medium | Faster than incr |
| Continuous (WAL) | Seconds | Stream | Point-in-time |

## Standby Models

| Model | RTO (Recovery Time Objective, допустимое время восстановления) | Cost | Data sync |
|-------|-----|------|-----------|
| Hot Standby | Minutes | $$$ | Sync replication |
| Warm Standby | Hours | $$ | Async, periodic |
| Cold Standby | Days | $ | Backups only |

## Active-Passive vs Active-Active

### Active-Passive DR (Disaster Recovery, аварийное восстановление)

- ➕ **Плюсы:** simpler; no write conflicts; cheaper.
- ➖ **Минусы / Цена:** failover time; idle standby cost.
- 📍 **Где:** most enterprise DR (Disaster Recovery, аварийное восстановление).

### Active-Active (Multi-Region)

- ➕ **Плюсы:** lowest RTO (Recovery Time Objective, допустимое время восстановления)/RPO (Recovery Point Objective, допустимая потеря данных); geo-low latency.
- ➖ **Минусы / Цена:** conflict resolution; 2x infra; complex routing (Route53, GeoDNS).
- 📍 **Где:** global apps (Netflix, Uber).

## Multi-AZ vs Multi-Region

| | Multi-AZ | Multi-Region |
|--|----------|--------------|
| Protects from | DC failure | Region failure |
| Latency | Same region | Cross-region |
| Cost | Moderate | High |

## Sync vs Async Cross-Region Replication

- **Sync:** RPO (Recovery Point Objective, допустимая потеря данных) ≈ 0, latency penalty on every write.
- **Async:** RPO (Recovery Point Objective, допустимая потеря данных) > 0 (seconds-minutes), better write latency.

**На шаге 2:** RPO (Recovery Point Objective, допустимая потеря данных)/RTO (Recovery Time Objective, допустимое время восстановления) targets. **На шаге 5:** pattern. **На шаге 6:** AWS RDS cross-region, etc.


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

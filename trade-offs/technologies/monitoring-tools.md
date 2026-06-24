---
layer: product
steps: [6]
related:
  - architecture/observability-architecture
---

# Monitoring Tools (инструменты мониторинга)

> **Главное:** Monitoring tools — metrics, logs, tracing choices. Вход — SLOs and debug needs.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need long-term metrics? | Да | Prometheus + remote |
| Need traces? | Да | Jaeger / Tempo |

## Цепочка решений

Deep Dive §4.x pattern → Deep Dive §4.x (tech)

## Prometheus + Grafana (OSS)

- ➕ Free; pull model; PromQL; K8s (Kubernetes, оркестратор контейнеров) native; huge exporter ecosystem.
- ➖ Self-hosted ops; long-term storage needs Thanos/Cortex; no built-in APM.
- 📍 **Where:** K8s (Kubernetes, оркестратор контейнеров), cost-conscious, engineering-heavy ops.

## Datadog / New Relic (Commercial SaaS)

- ➕ Full stack: metrics + logs + traces + RUM; low setup; ML (Machine Learning, машинное обучение) alerting.
- ➖ $$$ at scale; vendor lock-in; bill shock risk.
- 📍 **Where:** fast time-to-observability, enterprise, on-call maturity.

## ELK / OpenSearch (Logs)

- ➕ Powerful log search; Kibana dashboards; self-hosted or managed.
- ➖ Resource hungry; ops complexity; licensing (Elastic vs OpenSearch fork).
- 📍 **Where:** log-centric debugging, compliance audit trails.

## Jaeger / Tempo (Traces)

- ➕ OpenTelemetry compatible; distributed trace visualization.
- ➖ Needs instrumentation in all services; storage cost at high volume.
- 📍 **Where:** microservices latency debugging.

## CloudWatch (AWS)

- ➕ Zero setup on AWS; integrated alarms; Logs Insights.
- ➖ Weak cross-cloud; query UX (User Experience, пользовательский опыт); cost unpredictable.
- 📍 **Where:** AWS-only, minimal ops.

## Comparison (сравнение)

| Priority | Pick |
|----------|------|
| Cost + K8s (Kubernetes, оркестратор контейнеров) | Prometheus + Grafana + Loki |
| Speed to prod | Datadog |
| AWS-only simple | CloudWatch |
| Log search focus | ELK/OpenSearch |


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

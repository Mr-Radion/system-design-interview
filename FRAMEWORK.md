# Framework System Design

Фреймворк для прохождения system design на собесе.

## Workflow

| Шаг | Файл |
|-----|------|
| 1. Функциональные требования | [workflow/01-functional-requirements.md](workflow/01-functional-requirements.md) |
| 2. NFR + trade-offs | [workflow/02-non-functional-requirements.md](workflow/02-non-functional-requirements.md) |
| 3. API | [workflow/03-api-design.md](workflow/03-api-design.md) |
| 4. Data model | [workflow/04-data-model.md](workflow/04-data-model.md) |
| 5. High-Level Design | [workflow/05-high-level-design.md](workflow/05-high-level-design.md) |

## Пример целиком

[examples/instagram-feed.md](examples/instagram-feed.md) — лента постов (Instagram-like), все 5 шагов.

## Trade-offs

45 тем. Группировка = чеклист [шага 2](workflow/02-non-functional-requirements.md); API / Data / HLD — шаги 3–5.

### Шаг 2 — по блокам NFR

#### Performance

| Тема | Файл |
|------|------|
| Latency vs throughput | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| Performance vs cost | [performance-vs-cost](trade-offs/constraints/performance-vs-cost.md) |
| CDN + object storage | [cdn-object-storage-pattern](trade-offs/architecture/cdn-object-storage-pattern.md) |

#### Scalability

| Тема | Файл |
|------|------|
| Vertical vs horizontal | [vertical-vs-horizontal-scaling](trade-offs/constraints/vertical-vs-horizontal-scaling.md) |
| Scalability vs performance | [scalability-vs-performance](trade-offs/constraints/scalability-vs-performance.md) |
| Autoscaling vs fixed | [autoscaling-vs-fixed-capacity](trade-offs/constraints/autoscaling-vs-fixed-capacity.md) |
| Caching patterns | [caching-patterns](trade-offs/architecture/caching-patterns.md) |

#### Consistency

| Тема | Файл |
|------|------|
| CAP / PACELC | [cap-pacelc-distributed](trade-offs/architecture/cap-pacelc-distributed.md) |
| Consistency как NFR | [consistency-as-nfr](trade-offs/constraints/consistency-as-nfr.md) |

#### Reliability

| Тема | Файл |
|------|------|
| SLA, RPO/RTO | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| Disaster recovery | [disaster-recovery-pattern](trade-offs/architecture/disaster-recovery-pattern.md) |
| Resilience & backpressure | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |

#### Observability

| Тема | Файл |
|------|------|
| Metrics, logs, traces | [observability-architecture](trade-offs/architecture/observability-architecture.md) |
| Monitoring tools | [monitoring-tools](trade-offs/technologies/monitoring-tools.md) |

#### Processing

| Тема | Файл |
|------|------|
| Batch vs stream | [batch-vs-stream](trade-offs/architecture/batch-vs-stream.md) |

#### Security

Отдельных файлов нет — auth, rate limit, signed URL фиксируем в NFR и [API gateways](trade-offs/technologies/api-gateways.md).

#### Infra (итог)

| Тема | Файл |
|------|------|
| Выбор технологий | [technology-selection-meta](trade-offs/technologies/technology-selection-meta.md) |
| Databases | [databases](trade-offs/technologies/databases.md) |
| Caches | [caches](trade-offs/technologies/caches.md) |
| Message brokers | [message-brokers](trade-offs/technologies/message-brokers.md) |
| Object storage | [object-storage](trade-offs/technologies/object-storage.md) |
| API gateways | [api-gateways](trade-offs/technologies/api-gateways.md) |
| Load balancers | [load-balancers-proxies](trade-offs/technologies/load-balancers-proxies.md) |

### Шаг 3 — API

| Тема | Файл |
|------|------|
| REST vs gRPC vs GraphQL | [rest-grpc-graphql](trade-offs/api/rest-grpc-graphql.md) |
| Sync vs async | [sync-async-messaging](trade-offs/api/sync-async-messaging.md) |
| RPC vs queue | [rpc-vs-queue](trade-offs/api/rpc-vs-queue.md) |
| Push vs pull | [push-vs-pull-delivery](trade-offs/api/push-vs-pull-delivery.md) |
| Real-time transport | [realtime-transport](trade-offs/api/realtime-transport.md) |
| Write idempotency | [write-api-idempotency](trade-offs/api/write-api-idempotency.md) |
| API versioning | [api-versioning-evolution](trade-offs/api/api-versioning-evolution.md) |
| Pagination | [pagination-cursor-offset](trade-offs/data/pagination-cursor-offset.md) |

### Шаг 4 — Data

| Тема | Файл |
|------|------|
| SQL vs NoSQL | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |
| Norm vs denorm | [normalization-denormalization](trade-offs/data/normalization-denormalization.md) |
| Indexing | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| Sharding vs partition | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Replication sync/async | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Master-slave vs multi-master | [master-slave-multi-master](trade-offs/data/master-slave-multi-master.md) |

### Шаг 5 — High-Level Design

| Тема | Файл |
|------|------|
| Monolith vs micro | [monolith-microservices](trade-offs/architecture/monolith-microservices.md) |
| Stateless vs stateful | [stateless-stateful](trade-offs/architecture/stateless-stateful.md) |
| Load balancing L4/L7 | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| Messaging | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| Saga vs outbox | [saga-vs-outbox](trade-offs/architecture/saga-vs-outbox.md) |
| Orchestration vs choreography | [orchestration-choreography-saga](trade-offs/architecture/orchestration-choreography-saga.md) |
| Concurrency vs parallelism | [concurrency-vs-parallelism](trade-offs/architecture/concurrency-vs-parallelism.md) |
| Distributed coordination | [distributed-coordination](trade-offs/architecture/distributed-coordination.md) |
| ETL pipeline | [etl-pipeline-pattern](trade-offs/architecture/etl-pipeline-pattern.md) |

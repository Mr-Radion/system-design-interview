# Glossary (глоссарий)

Bilingual RU/EN — ключевые термины по [47 trade-offs](FRAMEWORK.md#trade-offs). Разделы: [NFR](#nfr) · [API](#api) · [Data](#data) · [Architecture](#architecture) · [Technologies](#technologies) · [Формулы](#formulas)

---

## NFR и ограничения {#nfr}

| Термин RU | EN | Кратко | Trade-off |
|-----------|-----|--------|-----------|
| Задержка | Latency | Время ответа одного запроса | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| Пропускная способность | Throughput | Запросов/сек или bytes/сек | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| Производительность vs стоимость | Performance vs cost | Больше железа/реплик = быстрее, но дороже | [performance-vs-cost](trade-offs/constraints/performance-vs-cost.md) |
| Вертикальное масштабирование | Vertical scaling | Больше CPU/RAM на одной ноде | [vertical-vs-horizontal-scaling](trade-offs/constraints/vertical-vs-horizontal-scaling.md) |
| Горизонтальное масштабирование | Horizontal scaling | Больше нод: sharding, stateless app + LB, cache — **не DB replication** | [vertical-vs-horizontal-scaling](trade-offs/constraints/vertical-vs-horizontal-scaling.md) |
| Read offload vs HA replication | Read offload vs HA replication | Replica read — bonus с lag; primary цель repl — failover/DR | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Масштабируемость vs производительность | Scalability vs performance | Добавление нод ≠ линейный выигрыш на одном запросе | [scalability-vs-performance](trade-offs/constraints/scalability-vs-performance.md) |
| Автоскейлинг | Autoscaling | Elastic capacity по метрикам | [autoscaling-vs-fixed-capacity](trade-offs/constraints/autoscaling-vs-fixed-capacity.md) |
| Согласованность как NFR | Consistency as NFR | Strong vs eventual — явное требование к операции | [consistency-as-nfr](trade-offs/constraints/consistency-as-nfr.md) |
| SLA | Service Level Agreement | Договорённый uptime/latency | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| SLO | Service Level Objective | Внутренняя цель (p99 ≤ X ms) | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| RPO | Recovery Point Objective | Сколько данных можно потерять при сбое | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| RTO | Recovery Time Objective | За сколько восстановить сервис | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |

---

## API {#api}

| Термин RU | EN | Кратко | Trade-off |
|-----------|-----|--------|-----------|
| REST | Representational State Transfer | Resource + HTTP verbs, JSON | [rest-grpc-graphql](trade-offs/api/rest-grpc-graphql.md) |
| gRPC | gRPC | Binary protobuf, streaming, internal RPC | [rest-grpc-graphql](trade-offs/api/rest-grpc-graphql.md) |
| GraphQL | GraphQL | Клиент выбирает поля; N+1 риск | [rest-grpc-graphql](trade-offs/api/rest-grpc-graphql.md) |
| Синхронный вызов | Sync call | Клиент ждёт ответ | [sync-async-messaging](trade-offs/api/sync-async-messaging.md) |
| Асинхронная обработка | Async processing | ACK быстро, работа в фоне | [sync-async-messaging](trade-offs/api/sync-async-messaging.md) |
| RPC | Remote Procedure Call | Request-response между сервисами | [rpc-vs-queue](trade-offs/api/rpc-vs-queue.md) |
| Очередь задач | Task queue | At-least-once, worker pool | [rpc-vs-queue](trade-offs/api/rpc-vs-queue.md) |
| Push-доставка | Push delivery | Сервер шлёт клиенту (WebSocket) | [push-vs-pull-delivery](trade-offs/api/push-vs-pull-delivery.md) |
| Pull / polling | Pull delivery | Клиент сам опрашивает | [push-vs-pull-delivery](trade-offs/api/push-vs-pull-delivery.md) |
| WebSocket | WebSocket | Duplex real-time channel | [realtime-transport](trade-offs/api/realtime-transport.md) |
| SSE | Server-Sent Events | One-way server → client stream | [realtime-transport](trade-offs/api/realtime-transport.md) |
| Идемпотентность записи | Write idempotency | Повтор запроса = тот же результат | [write-api-idempotency](trade-offs/api/write-api-idempotency.md) |
| Idempotency-Key | Idempotency-Key | HTTP header для dedup на API | [write-api-idempotency](trade-offs/api/write-api-idempotency.md) |
| Версионирование API | API versioning | URL / header / contract evolution | [api-versioning-evolution](trade-offs/api/api-versioning-evolution.md) |
| Cursor pagination | Cursor pagination | Stable page по opaque token | [pagination-cursor-offset](trade-offs/data/pagination-cursor-offset.md) |
| Offset pagination | Offset pagination | `LIMIT/OFFSET`; плохо на больших offset | [pagination-cursor-offset](trade-offs/data/pagination-cursor-offset.md) |

---

## Data — хранение и индексы {#data}

| Термин RU | EN | Кратко | Trade-off |
|-----------|-----|--------|-----------|
| OLTP | Online Transaction Processing | Короткие TX, row-oriented | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |
| OLAP | Online Analytical Processing | Агрегации, column store | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |
| <a id="htap"></a> HTAP | Hybrid Transactional/Analytical Processing | OLTP + analytics без тяжёлого ETL | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |
| Нормализация | Normalization | Меньше дублирования, JOIN | [normalization-denormalization](trade-offs/data/normalization-denormalization.md) |
| Денормализация | Denormalization | Дубли для read performance | [normalization-denormalization](trade-offs/data/normalization-denormalization.md) |
| B-Tree индекс | B-Tree index | Default: range + equality | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| GIN / inverted index | GIN / inverted index | Full-text, JSONB, arrays | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| LSM-tree | Log-Structured Merge-tree | Write-heavy, level compaction | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| HNSW | Hierarchical NSW | ANN для vector search | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| IVFFlat | Inverted File Flat | Vector clusters + probe | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| Composite index | Composite index | Multi-column key order matters | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| Шардирование | Sharding | Горизонтальный split данных | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Партиционирование | Partitioning | Split внутри одной БД | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Consistent hashing | Consistent hashing | Минимум remapping при add/remove node | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| <a id="rendezvous-hashing"></a> Rendezvous hashing | Rendezvous / highest random weight | Меньше skew vs naive hash | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Горячий ключ | Hot key | Один key/shard — непропорциональный трафик | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Rebalancing | Rebalancing | Перераспределение данных между shards | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Синхронная репликация | Sync replication | Commit после ACK replica | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Асинхронная репликация | Async replication | Commit на primary, replica догоняет | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Semi-sync replication | Semi-sync replication | Ждём 1 replica, остальные async | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Master-slave | Master-slave / primary-replica | Один writer, N readers | [master-slave-multi-master](trade-offs/data/master-slave-multi-master.md) |
| Multi-master | Multi-master | Несколько writers, conflict resolution | [master-slave-multi-master](trade-offs/data/master-slave-multi-master.md) |
| Double-entry ledger | Double-entry bookkeeping | Debit + credit парами | [paypal-payments](examples/paypal-payments.md) |
| Materialized view | Materialized view | Snapshot тяжёлого запроса; refresh по расписанию | [normalization-denormalization](trade-offs/data/normalization-denormalization.md) |
| Snowflake ID | Snowflake ID | 64-bit sortable ID (time + region) | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| ULID | Universally Unique Lexicographically Sortable Identifier | UUID, упорядоченный по времени | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Connection pool | Connection pool | PgBouncer, HikariCP — must-have highload | [databases](trade-offs/technologies/databases.md) |
| Отложенная транзакция | Deferrable transaction | TX может ждать lock (PostgreSQL) | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |

---

## Architecture — компоненты и паттерны {#architecture}

| Термин RU | EN | Кратко | Trade-off |
|-----------|-----|--------|-----------|
| CAP | CAP theorem | C/A/P — pick 2 в partition | [cap-pacelc-distributed](trade-offs/architecture/cap-pacelc-distributed.md) |
| PACELC | PACELC | Если partition → CAP; иначе Latency vs Consistency | [cap-pacelc-distributed](trade-offs/architecture/cap-pacelc-distributed.md) |
| CDN | Content Delivery Network | Edge cache для static/media | [cdn-object-storage-pattern](trade-offs/architecture/cdn-object-storage-pattern.md) |
| Cache-aside | Cache-aside | App читает cache → miss → DB → fill | [caching-patterns](trade-offs/architecture/caching-patterns.md) |
| Read-through | Read-through cache | Cache сам грузит из DB | [caching-patterns](trade-offs/architecture/caching-patterns.md) |
| Write-through | Write-through cache | Write в cache + DB sync | [caching-patterns](trade-offs/architecture/caching-patterns.md) |
| Write-behind | Write-behind / write-back | Write в cache, async flush в DB | [caching-patterns](trade-offs/architecture/caching-patterns.md) |
| Алгоритм вытеснения | Eviction policy | LRU, LFU, 2Q — что удалять | [cache-eviction-policies](trade-offs/architecture/cache-eviction-policies.md) |
| LRU | Least Recently Used | Вытесняем давно не использованное | [cache-eviction-policies](trade-offs/architecture/cache-eviction-policies.md) |
| LFU | Least Frequently Used | Вытесняем редко используемое | [cache-eviction-policies](trade-offs/architecture/cache-eviction-policies.md) |
| Cache miss attack | Cache miss attack | Намеренный miss → overload DB | [cache-eviction-policies](trade-offs/architecture/cache-eviction-policies.md) |
| L4 балансировка | L4 load balancing | TCP/UDP, IP:port | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| L7 балансировка | L7 load balancing | HTTP path/header routing | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| <a id="power-of-two-choices"></a> Power of two choices | Power of two choices | 2 random backend → менее loaded | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| GeoDNS | GeoDNS | DNS routing к ближайшему DC | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| Монолит | Monolith | Один deployable | [monolith-microservices](trade-offs/architecture/monolith-microservices.md) |
| Микросервисы | Microservices | Независимые сервисы + сеть | [monolith-microservices](trade-offs/architecture/monolith-microservices.md) |
| Stateless | Stateless service | Состояние в DB/cache, pod replaceable | [stateless-stateful](trade-offs/architecture/stateless-stateful.md) |
| Stateful | Stateful service | Локальное состояние на ноде | [stateless-stateful](trade-offs/architecture/stateless-stateful.md) |
| Pub/sub | Publish-subscribe | Topic, many consumers | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| Event Notification | Event Notification | Событие = id, consumer тянет state | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| Event-Carried State Transfer | ECST | Полный state в payload | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| <a id="event-collaboration"></a> Event Collaboration | Event Collaboration | Shared workflow через события | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| Saga | Saga | Распределённая TX через локальные TX + compensate | [saga-vs-outbox](trade-offs/architecture/saga-vs-outbox.md) |
| Transactional outbox | Transactional outbox | Outbox row в той же TX что бизнес-данные | [saga-vs-outbox](trade-offs/architecture/saga-vs-outbox.md) |
| Orchestration | Orchestration | Central coordinator saga steps | [orchestration-choreography-saga](trade-offs/architecture/orchestration-choreography-saga.md) |
| Choreography | Choreography | Сервисы реагируют на события без центра | [orchestration-choreography-saga](trade-offs/architecture/orchestration-choreography-saga.md) |
| Компенсирующая TX | Compensating transaction | Undo через обратную операцию, не DELETE | [saga-vs-outbox](trade-offs/architecture/saga-vs-outbox.md) |
| Concurrency | Concurrency | Много задач, один core (interleaving) | [concurrency-vs-parallelism](trade-offs/architecture/concurrency-vs-parallelism.md) |
| Parallelism | Parallelism | Одновременное выполнение на N cores | [concurrency-vs-parallelism](trade-offs/architecture/concurrency-vs-parallelism.md) |
| Leader election | Leader election | Один coordinator в cluster | [distributed-coordination](trade-offs/architecture/distributed-coordination.md) |
| Алгоритм забияки | Bully election | «Старший» ID → leader | [distributed-coordination](trade-offs/architecture/distributed-coordination.md) |
| Distributed lock | Distributed lock | Mutex через Redis/ZK/etcd | [distributed-coordination](trade-offs/architecture/distributed-coordination.md) |
| ETL | Extract-Transform-Load | Batch pipeline в DWH | [etl-pipeline-pattern](trade-offs/architecture/etl-pipeline-pattern.md) |
| Batch processing | Batch processing | Периодические job'ы | [batch-vs-stream](trade-offs/architecture/batch-vs-stream.md) |
| Stream processing | Stream processing | Continuous over event log | [batch-vs-stream](trade-offs/architecture/batch-vs-stream.md) |
| Circuit breaker | Circuit breaker | Fail fast при cascade | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |
| Backpressure | Backpressure | Slow down producer при overload | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |
| Rate limiting | Rate limiting | Token bucket / leaky bucket | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |
| Bulkhead | Bulkhead | Изоляция пулов ресурсов | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |
| Disaster recovery | Disaster recovery | Failover DC, backup restore | [disaster-recovery-pattern](trade-offs/architecture/disaster-recovery-pattern.md) |
| Active-passive DR | Active-passive DR | Standby DC включается при сбое | [disaster-recovery-pattern](trade-offs/architecture/disaster-recovery-pattern.md) |
| Rolling deploy | Rolling deployment | Постепенная замена pods | [deployment-release-strategies](trade-offs/architecture/deployment-release-strategies.md) |
| Blue-green | Blue-green deployment | Два окружения, switch traffic | [deployment-release-strategies](trade-offs/architecture/deployment-release-strategies.md) |
| Canary release | Canary release | % traffic на новую версию | [deployment-release-strategies](trade-offs/architecture/deployment-release-strategies.md) |
| Observability | Observability | Metrics + logs + traces | [observability-architecture](trade-offs/architecture/observability-architecture.md) |
| Distributed tracing | Distributed tracing | trace_id через сервисы | [observability-architecture](trade-offs/architecture/observability-architecture.md) |

---

## Technologies — что выбирать на §7 {#technologies}

| Термин RU | EN | Кратко | Trade-off |
|-----------|-----|--------|-----------|
| Object storage | Object storage | S3-compatible blobs | [object-storage](trade-offs/technologies/object-storage.md) |
| Presigned URL | Presigned URL | Client upload direct to S3 | [cdn-object-storage-pattern](trade-offs/architecture/cdn-object-storage-pattern.md) |
| Message broker | Message broker | Kafka / RabbitMQ / SQS | [message-brokers](trade-offs/technologies/message-brokers.md) |
| Event log | Event log | Ordered, replayable (Kafka) | [message-brokers](trade-offs/technologies/message-brokers.md) |
| API Gateway | API Gateway | Auth, rate limit, routing | [api-gateways](trade-offs/technologies/api-gateways.md) |
| Reverse proxy | Reverse proxy | Nginx, Envoy front backends | [load-balancers-proxies](trade-offs/technologies/load-balancers-proxies.md) |
| Wide-column store | Wide-column store | Cassandra, Scylla — time-series writes | [databases](trade-offs/technologies/databases.md) |
| In-memory cache | In-memory cache | Redis, Memcached | [caches](trade-offs/technologies/caches.md) |
| Technology selection | Technology selection | Decision tree после HLD | [technology-selection-meta](trade-offs/technologies/technology-selection-meta.md) |

---

## Быстрые формулы (собес) {#formulas}

| Формула | Смысл |
|---------|-------|
| `AvgTime = DB_time × miss_rate + cache_time × hit_rate` | Средняя latency с cache |
| `Peak RPS = DAU × actions/day ÷ 86_400 × peak_factor` | Back-of-envelope throughput |
| `Storage = rows × row_size × retention` | Disk planning |
| `Replication lag ≤ read staleness SLO` | Когда replica OK для read |

→ latency порядки величин: [workflow/02](workflow/02-non-functional-requirements.md)

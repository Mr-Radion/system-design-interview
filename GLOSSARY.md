# Glossary (глоссарий)

RU/EN термины для system design на собесе. Формат таблицы:

| Колонка | Смысл |
|---------|--------|
| **Термин RU** | как пишешь на доске |
| **EN · расшифровка** | полное имя + перевод |
| **Пояснение** | что это, зачем на собесе, пример |
| **Trade-off** | куда копать глубже |

Разделы: [§2.1 параметры](#step2-metrics) · [NFR](#nfr) · [API](#api) · [Data](#data) · [Architecture](#architecture) · [Technologies](#technologies) · [Формулы](#formulas)

→ таблица вопросов и расчётов: [workflow/02 §2.1](workflow/02-non-functional-requirements.md#21-цифры-на-доску)

---

## §2.1 — параметры на доске {#step2-metrics}

| Термин RU | EN · расшифровка | Пояснение | Trade-off |
|-----------|------------------|-----------|-----------|
| <a id="dau"></a> DAU | DAU (Daily Active Users — дневная аудитория) | Сколько **уникальных пользователей за день** хотя бы раз открыли продукт. На собесе — главный вход для QPS: «50M DAU × 5 reads/day ÷ 86_400». | — |
| <a id="mau"></a> MAU | MAU (Monthly Active Users — месячная аудитория) | Уникальные пользователи за **30 дней**. DAU ≈ MAU × 25–40% для mobile; для B2B часто спрашивают registered, не MAU. | — |
| <a id="ccu"></a> CCU | CCU (Concurrent Users — одновременно онлайн) | Сколько пользователей **в один момент** в системе (игры, чат). Точнее DAU×10% для realtime; для MMO CCU — hard requirement. | — |
| Registered users | Registered users — зарегистрированные | Все аккаунты ever, не активность. DAU/MAU **≤** registered. Для расчётов бери DAU/CCU, не registered. | — |
| <a id="qps"></a> QPS / RPS | QPS (Queries Per Second) / RPS (Requests Per Second) | **Запросов в секунду** к API или БД. Read QPS = users × reads/day ÷ 86_400. На доске: «~2.9K read/s». | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| <a id="86400"></a> 86_400 | 86_400 seconds — секунд в сутках | 24×60×60. Делим дневную активность на это число → **средний** RPS. Пик = avg × burst (×3–×5). | — |
| <a id="p50"></a> p50 | p50 (50th percentile — медиана задержки) | **Половина** запросов быстрее этого времени. «Типичный» UX; на собесе реже целятся, чем p99. | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| <a id="p95"></a> p95 | p95 (95th percentile — 95-й перцентиль) | **95%** запросов быстрее; 5% медленнее. Промежуточная SLO: «95% < 500 ms». | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| <a id="p99"></a> p99 | p99 (99th percentile — 99-й перцентиль задержки) | **99%** запросов укладываются в это время, **1% самых медленных** — дольше. На собесе смотрят p99 **sync path**, не среднее: ловит «хвост» latency (cold cache, GC). | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| <a id="sync-path"></a> Sync path | Synchronous path — синхронный путь | Цепочка, где **пользователь ждёт ответ** (GET feed, POST transfer). SLO/p99 считают здесь. Async (fan-out, analytics) — отдельный budget. | [sync-async-messaging](trade-offs/api/sync-async-messaging.md) |
| <a id="headroom"></a> Headroom | Headroom — запас мощности | Capacity **выше пика** (×2–×3), чтобы пережить burst без деградации. «Peak 500K/s, design for ×2 consumers». | [vertical-vs-horizontal-scaling](trade-offs/constraints/vertical-vs-horizontal-scaling.md) |
| Peak factor / burst | Peak factor / burst — пиковый коэффициент | Во сколько раз **пик** больше среднего: prime time ×5, payday ×3, patch day ×10. Умножаешь avg QPS. | — |
| <a id="hot-path"></a> Hot path | Hot path — горячий путь | Самый частый и критичный сценарий (read feed, send message). **Bottleneck** обычно здесь — оптимизируешь первым. | — |
| <a id="bottleneck"></a> Bottleneck | Bottleneck — узкое место | Что **сломается первым** при росте нагрузки: bandwidth 20 GB/s, write 18.5K/s, dispatch 500K/s. Одна строка вывода §2.2 → §4.x. | — |
| Read:Write ratio | Read:Write ratio — соотношение чтений и записей | Сколько read на один write. Instagram ~25:1 → CDN/cache; messaging 1:1 → broker/partitions. | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| <a id="bandwidth"></a> Bandwidth | Bandwidth — пропускная способность сети | **Байт/сек** (GB/s, Mbps), не запросы. Read bandwidth = QPS × payload × items. Часто bottleneck для media/feed. | [cdn-object-storage-pattern](trade-offs/architecture/cdn-object-storage-pattern.md) |
| Strong consistency (CP) | Strong consistency — строгая согласованность | После write **все** видят одно значение сразу. Деньги, inventory, game progress → CP, RPO ≈ 0. | [consistency-as-nfr](trade-offs/constraints/consistency-as-nfr.md) |
| Eventual consistency | Eventual consistency — eventual согласованность | Реплики **догоняют** через ms–sec. OK для ленты, likes, DLR stats — если продукт допускает stale. | [consistency-as-nfr](trade-offs/constraints/consistency-as-nfr.md) |
| <a id="repl"></a> Replication (repl) | Replication — репликация | Копия данных на другую ноду. **Primary цель — HA/DR** (failover), не read scale. Read throughput → cache/CDN. | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| HA | HA (High Availability — высокая доступность) | Сервис переживает падение ноды/AZ без длительного outage. Multi-AZ, repl, health checks. | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| DR | DR (Disaster Recovery — аварийное восстановление) | План на падение **целого DC/региона**: backup, standby, RPO/RTO. Tier hot/warm/cold → §4.4. | [disaster-recovery-pattern](trade-offs/architecture/disaster-recovery-pattern.md) |

---

## NFR и ограничения {#nfr}

| Термин RU | EN · расшифровка | Пояснение | Trade-off |
|-----------|------------------|-----------|-----------|
| Задержка | Latency — задержка | **Сколько ms ждёт один запрос** от отправки до ответа. UX и SLO; не путать с throughput (сколько запросов/сек). | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| Пропускная способность | Throughput — пропускная способность | **Сколько работы в единицу времени**: req/s, msg/s, GB/s. Можно высокий throughput при высокой latency (batch). | [latency-vs-throughput](trade-offs/constraints/latency-vs-throughput.md) |
| Производительность vs стоимость | Performance vs cost | Больше CPU/replica/CDN = быстрее и надёжнее, но **$$**. На собесе называешь trade-off вслух. | [performance-vs-cost](trade-offs/constraints/performance-vs-cost.md) |
| Вертикальное масштабирование | Vertical scaling — scale up | **Больше RAM/CPU на одной машине**. Быстро, но потолок и single point of failure. | [vertical-vs-horizontal-scaling](trade-offs/constraints/vertical-vs-horizontal-scaling.md) |
| Горизонтальное масштабирование | Horizontal scaling — scale out | **Больше нод**: sharding, stateless app + LB, cache. Read scale — CDN/cache; **не** «ещё replica ради read». | [vertical-vs-horizontal-scaling](trade-offs/constraints/vertical-vs-horizontal-scaling.md) |
| Read offload vs HA replication | Read offload vs HA replication | Replica для read — **бонус** с replication lag. Repl ставят для **failover/RPO**, не как главный read scale. | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Масштабируемость vs производительность | Scalability vs performance | Больше нод ≠ каждый запрос быстрее. Scale-out помогает **суммарному** RPS, не latency одного hop. | [scalability-vs-performance](trade-offs/constraints/scalability-vs-performance.md) |
| Автоскейлинг | Autoscaling — автомасштабирование | Добавление/снятие инстансов по CPU/RPS/lag. Lag на scale-up — учитывай в burst. | [autoscaling-vs-fixed-capacity](trade-offs/constraints/autoscaling-vs-fixed-capacity.md) |
| Согласованность как NFR | Consistency as NFR | Явно фиксируешь: **strong** (money) или **eventual** (feed OK stale). Влияет на CAP, repl, saga. | [consistency-as-nfr](trade-offs/constraints/consistency-as-nfr.md) |
| <a id="sla"></a> SLA | SLA (Service Level Agreement — соглашение об уровне сервиса) | **Обещание клиенту** в контракте: uptime 99.9%, max latency. Нарушение → штрафы. 99.9% ≈ **8.7 ч** простоя/год. | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| <a id="sli"></a> SLI | SLI (Service Level Indicator — индикатор уровня сервиса) | **Конкретная метрика**, которую меряем: «p99 latency GET /feed», «error rate < 0.1%». Строится из logs/metrics/traces. | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| <a id="slo"></a> SLO | SLO (Service Level Objective — цель уровня сервиса) | **Внутренняя цель команды/SRE**, обычно **строже SLA**: «95% запросов < 500 ms» или «p99 ≤ 200 ms». SLI + порог; breach → алерт и фикс **до** эскалации к SLA. | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| <a id="rpo"></a> RPO | RPO (Recovery Point Objective — цель точки восстановления) | **Сколько данных можно потерять** при катастрофе. RPO ≈ 0 = нельзя потерять save/payment; RPO 1 h = OK потерять час логов. | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |
| <a id="rto"></a> RTO | RTO (Recovery Time Objective — цель времени восстановления) | **За сколько минут/часов** сервис снова работает после падения DC. Hot tier: RTO < 5 min; cold: дни restore. | [availability-slo-rpo-rto](trade-offs/constraints/availability-slo-rpo-rto.md) |

---

## API {#api}

| Термин RU | EN · расшифровка | Пояснение | Trade-off |
|-----------|------------------|-----------|-----------|
| REST | REST (Representational State Transfer) | HTTP + JSON, ресурсы (`/users/{id}`), verbs GET/POST. **Default для public/mobile API** на собесе. | [rest-grpc-graphql](trade-offs/api/rest-grpc-graphql.md) |
| gRPC | gRPC (Google Remote Procedure Call) | Бинарный protobuf, streaming, **internal** service-to-service. Быстрее REST, сложнее для browser. | [rest-grpc-graphql](trade-offs/api/rest-grpc-graphql.md) |
| GraphQL | GraphQL | Клиент **выбирает поля** одним запросом; риск N+1 и тяжёлых resolvers без DataLoader. | [rest-grpc-graphql](trade-offs/api/rest-grpc-graphql.md) |
| Синхронный вызов | Sync call — синхронный вызов | Клиент **блокируется** до ответа. User-facing hot path; latency budget = sum hops. | [sync-async-messaging](trade-offs/api/sync-async-messaging.md) |
| Асинхронная обработка | Async processing — асинхронная обработка | Быстрый ACK (202), работа в **очереди/worker**. Fan-out, AI scan, DLR — не держишь HTTP open. | [sync-async-messaging](trade-offs/api/sync-async-messaging.md) |
| RPC | RPC (Remote Procedure Call — удалённый вызов процедуры) | «Вызов функции» по сети: request → response. gRPC/Thrift; sync coupling между сервисами. | [rpc-vs-queue](trade-offs/api/rpc-vs-queue.md) |
| Очередь задач | Task queue — очередь задач | Producer кладёт job, workers забирают. **At-least-once**, retry, DLQ. BullMQ, SQS, Rabbit task queues. | [rpc-vs-queue](trade-offs/api/rpc-vs-queue.md) |
| Push-доставка | Push delivery — push-доставка | Сервер **сам шлёт** клиенту (WebSocket, push notification). Chat, live scores — без polling. | [push-vs-pull-delivery](trade-offs/api/push-vs-pull-delivery.md) |
| Pull / polling | Pull delivery — pull / опрос | Клиент **сам спрашивает** «есть новое?». Проще, но latency и лишний трафик при частом poll. | [push-vs-pull-delivery](trade-offs/api/push-vs-pull-delivery.md) |
| WebSocket | WebSocket | **Дуплексный** канал поверх HTTP upgrade. Real-time chat; stateful connections → LB sticky. | [realtime-transport](trade-offs/api/realtime-transport.md) |
| SSE | SSE (Server-Sent Events — события с сервера) | **One-way** stream server→client over HTTP. Проще WebSocket для live feed/notifications. | [realtime-transport](trade-offs/api/realtime-transport.md) |
| Идемпотентность записи | Write idempotency — идемпотентность записи | **Повтор того же запроса** не создаёт дубль (double charge, duplicate grant). Критично для payments/webhooks. | [write-api-idempotency](trade-offs/api/write-api-idempotency.md) |
| Idempotency-Key | Idempotency-Key — ключ идемпотентности | HTTP header с **уникальным id** операции; сервер dedup по key 24h. Stripe/PayPal pattern. | [write-api-idempotency](trade-offs/api/write-api-idempotency.md) |
| Версионирование API | API versioning — версионирование API | `/v1/`, header `Accept-Version` — **не ломать** старых клиентов при эволюции контракта. | [api-versioning-evolution](trade-offs/api/api-versioning-evolution.md) |
| Cursor pagination | Cursor pagination — курсорная пагинация | Стабильная следующая страница по **opaque token** (last_id). Нет skip cost как у OFFSET. | [pagination-cursor-offset](trade-offs/data/pagination-cursor-offset.md) |
| Offset pagination | Offset pagination — offset-пагинация | `LIMIT/OFFSET` или page number. Просто, но **медленно** на больших offset (deep pages). | [pagination-cursor-offset](trade-offs/data/pagination-cursor-offset.md) |

---

## Data — хранение и индексы {#data}

| Термин RU | EN · расшифровка | Пояснение | Trade-off |
|-----------|------------------|-----------|-----------|
| OLTP | OLTP (Online Transaction Processing — операционная обработка) | Короткие **транзакции**, row store: orders, users, ledger. PostgreSQL default для core data. | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |
| OLAP | OLAP (Online Analytical Processing — аналитическая обработка) | **Агрегации** по большим объёмам, column store: ClickHouse, BigQuery. Reports, funnels — не hot path. | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |
| <a id="htap"></a> HTAP | HTAP (Hybrid Transactional/Analytical Processing) | OLTP + analytics **в одном** кластере без тяжёлого ETL. Редко на собесе; чаще split OLTP + OLAP. | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |
| Нормализация | Normalization — нормализация | Данные **без дублирования**, связи через FK и JOIN. Меньше anomaly, больше JOIN на read. | [normalization-denormalization](trade-offs/data/normalization-denormalization.md) |
| Денормализация | Denormalization — денормализация | **Дубли** полей в одной таблице/документе ради быстрого read (feed item embeds author name). | [normalization-denormalization](trade-offs/data/normalization-denormalization.md) |
| B-Tree индекс | B-Tree index — B-дерево | Default в SQL: **range** (`BETWEEN`) и equality (`=`). Primary key, `(user_id, created_at)`. | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| GIN / inverted index | GIN (Generalized Inverted Index) | Full-text, JSONB, arrays — **«содержит элемент»**. PostgreSQL GIN для search. | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| LSM-tree | LSM-tree (Log-Structured Merge-tree) | Write-optimized: memtable + **compaction**. RocksDB, Cassandra — high write throughput. | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| HNSW | HNSW (Hierarchical Navigable Small World) | ANN index для **vector search** (embeddings). Быстрый approximate nearest neighbor. | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| IVFFlat | IVFFlat (Inverted File Flat) | Vector index: **clusters + probe** k ближайших. Дешевле HNSW, хуже recall на больших данных. | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| Composite index | Composite index — составной индекс | Несколько колонок; **порядок важен**: `(a,b)` помогает `WHERE a=?`, не всегда `WHERE b=?`. | [indexing-strategy](trade-offs/data/indexing-strategy.md) |
| Шардирование | Sharding — шардирование | **Горизонтальный split** данных между несколькими БД/кластерами. Key: `hash(user_id)`. | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Партиционирование | Partitioning — партиционирование | Split **внутри одной** БД (by month, by hash). Управление retention, не cross-DB scale. | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Consistent hashing | Consistent hashing — consistent hashing | При add/remove ноды **минимум** ключей переезжает. Ring + virtual nodes для LB/shard routing. | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| <a id="rendezvous-hashing"></a> Rendezvous hashing | Rendezvous hashing — highest random weight | Каждый key выбирает shard с max hash(key,node). **Меньше skew**, чем naive mod N. | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Горячий ключ | Hot key — горячий ключ | Один key/shard получает **непропорциональный** трафик (celebrity, viral post). Решение: split key, local cache. | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Rebalancing | Rebalancing — ребалансировка | Перенос данных между shards при **росте** кластера. Downtime/cost — планируешь заранее. | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Синхронная репликация | Sync replication — синхронная репликация | Commit **после ACK** replica. RPO ≈ 0, но latency write ↑. CP ledger, semi-sync. | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Асинхронная репликация | Async replication — асинхронная репликация | Commit на **primary**, replica догоняет с lag. Быстрее write; возможна потеря последних секунд. | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Semi-sync replication | Semi-sync replication — полусинхронная | Ждём **одну** replica ACK, остальные async. Баланс RPO и latency. | [replication-sync-async](trade-offs/data/replication-sync-async.md) |
| Master-slave | Primary-replica (master-slave) | **Один writer**, N read replicas. Split-brain защита; multi-master сложнее. | [master-slave-multi-master](trade-offs/data/master-slave-multi-master.md) |
| Multi-master | Multi-master — multi-master | **Несколько writers**; нужен conflict resolution (LWW, CRDT). Geo-write, rare on собес. | [master-slave-multi-master](trade-offs/data/master-slave-multi-master.md) |
| Double-entry ledger | Double-entry bookkeeping — двойная запись | Каждая операция = **debit + credit** пары; sum(balance)=0. CP payments invariant. | [paypal-payments](examples/paypal-payments.md) |
| Materialized view | Materialized view — материализованное представление | **Snapshot** тяжёлого SELECT; refresh cron/trigger. Read fast, stale until refresh. | [normalization-denormalization](trade-offs/data/normalization-denormalization.md) |
| Snowflake ID | Snowflake ID | 64-bit **sortable** ID: timestamp + machine + sequence. Twitter IDs; shard-friendly. | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| ULID | ULID (Universally Unique Lexicographically Sortable Identifier) | UUID-подобный, **упорядочен по времени**. URL-safe, good for logs/keys. | [sharding-partitioning](trade-offs/data/sharding-partitioning.md) |
| Connection pool | Connection pool — пул соединений | Переиспользование DB connections (PgBouncer). **Must-have** highload — иначе TCP/handshake bottleneck. | [databases](trade-offs/technologies/databases.md) |
| Отложенная транзакция | Deferrable transaction — отложенная транзакция | TX может **ждать** lock до commit (PostgreSQL DEFERRABLE). Graph inserts без deadlock order. | [sql-vs-nosql-paradigm](trade-offs/data/sql-vs-nosql-paradigm.md) |

---

## Architecture — компоненты и паттерны {#architecture}

| Термин RU | EN · расшифровка | Пояснение | Trade-off |
|-----------|------------------|-----------|-----------|
| CAP | CAP theorem — теорема CAP | При **network partition** выбираешь 2 из 3: Consistency, Availability, Partition tolerance. На собесе — **по участку** системы. | [cap-pacelc-distributed](trade-offs/architecture/cap-pacelc-distributed.md) |
| PACELC | PACELC | **Если partition** → CAP; **иначе** trade-off Latency vs Consistency (async repl для L). | [cap-pacelc-distributed](trade-offs/architecture/cap-pacelc-distributed.md) |
| CDN | CDN (Content Delivery Network — сеть доставки контента) | **Edge cache** ближе к user для static/media. Read bandwidth 20 GB/s → CDN first. | [cdn-object-storage-pattern](trade-offs/architecture/cdn-object-storage-pattern.md) |
| Cache-aside | Cache-aside — cache-aside | App: read cache → **miss** → DB → fill cache. Самый частый pattern на собесе. | [caching-patterns](trade-offs/architecture/caching-patterns.md) |
| Read-through | Read-through cache | Cache **сам** грузит из DB при miss; app видит только cache API. | [caching-patterns](trade-offs/architecture/caching-patterns.md) |
| Write-through | Write-through cache | Write в cache **и** DB синхронно. Consistency проще, write latency ↑. | [caching-patterns](trade-offs/architecture/caching-patterns.md) |
| Write-behind | Write-behind (write-back) | Write в cache, **async flush** в DB. Быстрый write, риск потери при crash. | [caching-patterns](trade-offs/architecture/caching-patterns.md) |
| Алгоритм вытеснения | Eviction policy — политика вытеснения | Что **удалять** при full cache: LRU (давно не трогали), LFU (редко использовали). | [cache-eviction-policies](trade-offs/architecture/cache-eviction-policies.md) |
| LRU | LRU (Least Recently Used — давно не использовали) | Вытесняем ключ, к которому **давно не обращались**. Default для many caches. | [cache-eviction-policies](trade-offs/architecture/cache-eviction-policies.md) |
| LFU | LFU (Least Frequently Used — редко использовали) | Вытесняем **редко** запрашиваемые ключи. Лучше при hot subset (popular posts). | [cache-eviction-policies](trade-offs/architecture/cache-eviction-policies.md) |
| Cache miss attack | Cache miss attack — атака cache miss | Злоумышленник шлёт **уникальные keys** → 100% miss → DB overload. Защита: rate limit, TTL cap. | [cache-eviction-policies](trade-offs/architecture/cache-eviction-policies.md) |
| <a id="l4"></a> L4 балансировка | L4 load balancing — балансировка уровня 4 | OSI **Transport**: TCP/UDP, IP:port. Быстро, не видит HTTP path. NLB. | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| <a id="l7"></a> L7 балансировка | L7 load balancing — балансировка уровня 7 | OSI **Application**: HTTP path, headers, cookies. ALB/NGINX — routing `/api` vs `/static`. | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| <a id="power-of-two-choices"></a> Power of two choices | Power of two choices — «сила двух выборов» | LB пикает **2 random** backend, шлёт на **менее загруженный**. O(1) и меньше skew vs round-robin. | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| GeoDNS | GeoDNS — гео-DNS | DNS отдаёт IP **ближайшего** DC/regional edge. Latency ↓ для global users. | [load-balancing-l4-l7](trade-offs/architecture/load-balancing-l4-l7.md) |
| Монолит | Monolith — монолит | **Один** deployable artifact, shared DB. Fast start; coupling растёт с командой. | [monolith-microservices](trade-offs/architecture/monolith-microservices.md) |
| Микросервисы | Microservices — микросервисы | **Независимые** сервисы, своя БД, сеть между ними. Scale team + module; ops сложнее. | [monolith-microservices](trade-offs/architecture/monolith-microservices.md) |
| Stateless | Stateless service — stateless сервис | Состояние в **DB/cache**, pod можно убить и заменить. Horizontal scale + rolling deploy. | [stateless-stateful](trade-offs/architecture/stateless-stateful.md) |
| Stateful | Stateful service — stateful сервис | **Локальное** состояние на ноде (session, shard owner). Sticky sessions или resharding. | [stateless-stateful](trade-offs/architecture/stateless-stateful.md) |
| Pub/sub | Pub/sub (Publish-Subscribe — публикация-подписка) | Publisher → **topic** → many subscribers. Fan-out feed, events. Kafka, SNS. | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| Event Notification | Event Notification — уведомление о событии | Событие = **id/type**; consumer сам тянет полный state. Loose coupling. | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| Event-Carried State Transfer | ECST (Event-Carried State Transfer) | **Полный state** в payload события — consumer не ходит в source DB. | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| <a id="event-collaboration"></a> Event Collaboration | Event Collaboration — событийная коллаборация | Несколько сервисов **координируют** workflow только через события, без central orchestrator. | [messaging-patterns](trade-offs/architecture/messaging-patterns.md) |
| Saga | Saga — сага | Распределённая TX = **цепочка локальных TX** + compensate при fail. Cross-shard P2P, orders. | [saga-vs-outbox](trade-offs/architecture/saga-vs-outbox.md) |
| Transactional outbox | Transactional outbox — транзакционный outbox | Outbox row в **той же DB TX** что бизнес-данные → worker публикует в broker. No lost events. | [saga-vs-outbox](trade-offs/architecture/saga-vs-outbox.md) |
| Orchestration | Orchestration — оркестрация | **Central coordinator** (saga orchestrator) говорит сервисам шаг 1,2,3. Видимость, single point. | [orchestration-choreography-saga](trade-offs/architecture/orchestration-choreography-saga.md) |
| Choreography | Choreography — хореография | Сервисы **реагируют на события** друг друга без центра. Проще старт, сложнее debug. | [orchestration-choreography-saga](trade-offs/architecture/orchestration-choreography-saga.md) |
| Компенсирующая TX | Compensating transaction — компенсирующая транзакция | **Undo** бизнес-операции (refund, cancel reserve), не DELETE row. Saga rollback step. | [saga-vs-outbox](trade-offs/architecture/saga-vs-outbox.md) |
| Concurrency | Concurrency — конкурентность | Много задач **чередуются** на одном core (interleaving). Async I/O, goroutines. | [concurrency-vs-parallelism](trade-offs/architecture/concurrency-vs-parallelism.md) |
| Parallelism | Parallelism — параллелизм | Задачи **одновременно** на N cores. CPU-bound batch, map-reduce workers. | [concurrency-vs-parallelism](trade-offs/architecture/concurrency-vs-parallelism.md) |
| Leader election | Leader election — выбор лидера | Один **coordinator** в cluster (who writes shard, who runs cron). Raft, ZK, etcd. | [distributed-coordination](trade-offs/architecture/distributed-coordination.md) |
| Алгоритм забияки | Bully election — алгоритм «забияки» | Node с **большим ID** становится leader при fail. Простой учебный алгоритм. | [distributed-coordination](trade-offs/architecture/distributed-coordination.md) |
| Distributed lock | Distributed lock — распределённый lock | Mutex через **Redis/ZK/etcd** для «только один worker». TTL + fencing token против split-brain. | [distributed-coordination](trade-offs/architecture/distributed-coordination.md) |
| ETL | ETL (Extract-Transform-Load) | Batch: **вытащить** → преобразовать → загрузить в DWH. Nightly reports, analytics. | [etl-pipeline-pattern](trade-offs/architecture/etl-pipeline-pattern.md) |
| Batch processing | Batch processing — пакетная обработка | Job **по расписанию** (cron): aggregates, archive. Не realtime; проще и дешевле. | [batch-vs-stream](trade-offs/architecture/batch-vs-stream.md) |
| Stream processing | Stream processing — потоковая обработка | **Continuous** обработка event log (Kafka → Flink). Real-time dashboards, lag alerts. | [batch-vs-stream](trade-offs/architecture/batch-vs-stream.md) |
| Circuit breaker | Circuit breaker — автоматический выключатель | При cascade fail **stop calling** downstream, fail fast + fallback. Resilience4j pattern. | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |
| Backpressure | Backpressure — обратное давление | При overload **замедлить producer** (block publish, 429) вместо OOM. Kafka pause consumer. | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |
| Rate limiting | Rate limiting — ограничение частоты | Token bucket / leaky bucket: **max N req/s** per user/IP/API key. Gateway layer. | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |
| Bulkhead | Bulkhead — переборка | **Изолированные пулы** (thread pool per dependency) — один slow API не съедает все threads. | [resilience-backpressure](trade-offs/architecture/resilience-backpressure.md) |
| Disaster recovery | Disaster recovery — аварийное восстановление | План на **total DC loss**: backup, standby region, runbook, drills. RPO/RTO drive tier. | [disaster-recovery-pattern](trade-offs/architecture/disaster-recovery-pattern.md) |
| Active-passive DR | Active-passive DR | **Standby DC** включается при сбое primary. Warm/hot standby по RTO. | [disaster-recovery-pattern](trade-offs/architecture/disaster-recovery-pattern.md) |
| Rolling deploy | Rolling deployment — rolling deploy | Pods **постепенно** заменяются новой версией. Zero downtime при health checks. | [deployment-release-strategies](trade-offs/architecture/deployment-release-strategies.md) |
| Blue-green | Blue-green deployment | **Два** полных окружения; switch traffic одним flip. Быстрый rollback. | [deployment-release-strategies](trade-offs/architecture/deployment-release-strategies.md) |
| Canary release | Canary release — канареечный релиз | **% traffic** на новую версию; мониторинг error/latency перед full rollout. | [deployment-release-strategies](trade-offs/architecture/deployment-release-strategies.md) |
| Observability | Observability — наблюдаемость | **Metrics + logs + traces** — понять состояние системы снаружи. SLO alerts, debug prod. | [observability-architecture](trade-offs/architecture/observability-architecture.md) |
| Distributed tracing | Distributed tracing — распределённая трассировка | **trace_id** через все hops (gateway→service→DB). Jaeger, Zipkin — где потеряли ms. | [observability-architecture](trade-offs/architecture/observability-architecture.md) |

---

## Technologies — что выбирать в Deep Dive §4.x {#technologies}

| Термин RU | EN · расшифровка | Пояснение | Trade-off |
|-----------|------------------|-----------|-----------|
| Object storage | Object storage — объектное хранилище | **Blob** по key (S3): photos, exports, static. Cheap $/GB; не для transactional queries. | [object-storage](trade-offs/technologies/object-storage.md) |
| Presigned URL | Presigned URL — presigned URL | Временная **signed ссылка** для client upload/download direct to S3 — API не проксирует bytes. | [cdn-object-storage-pattern](trade-offs/architecture/cdn-object-storage-pattern.md) |
| Message broker | Message broker — брокер сообщений | **Kafka / RabbitMQ / SQS**: decouple, buffer, fan-out. Имя продукта — только после trade-off gate §4. | [message-brokers](trade-offs/technologies/message-brokers.md) |
| Event log | Event log — журнал событий | **Ordered, replayable** log (Kafka topic). Source of truth для async pipeline, audit. | [message-brokers](trade-offs/technologies/message-brokers.md) |
| API Gateway | API Gateway — API-шлюз | **Единая точка входа**: auth, rate limit, routing, TLS termination. Kong, Envoy, AWS API GW. | [api-gateways](trade-offs/technologies/api-gateways.md) |
| Reverse proxy | Reverse proxy — reverse proxy | **Front** для backends: NGINX, Envoy — LB, SSL, cache static. Не путать с forward proxy. | [load-balancers-proxies](trade-offs/technologies/load-balancers-proxies.md) |
| Wide-column store | Wide-column store — wide-column БД | **Cassandra, Scylla**: write-heavy, time-series, tunable consistency. Messages at scale. | [databases](trade-offs/technologies/databases.md) |
| In-memory cache | In-memory cache — in-memory кеш | **Redis, Memcached**: hot keys, session, rate limit counters. Sub-ms read. | [caches](trade-offs/technologies/caches.md) |
| Technology selection | Technology selection — выбор технологии | **После** trade-off gate в §4: «нужен ordered log + replay» → Kafka, не «сразу Kafka». | [technology-selection-meta](trade-offs/technologies/technology-selection-meta.md) |

---

## Быстрые формулы (собес) {#formulas}

| Формула | Пояснение |
|---------|-----------|
| `AvgTime = DB_time × miss_rate + cache_time × hit_rate` | **Средняя latency** с cache: weighted sum hit vs miss. Пример: 90% hit × 1 ms + 10% miss × 50 ms. |
| `Peak RPS = DAU × actions/day ÷ 86_400 × peak_factor` | **Back-of-envelope QPS**: подставь DAU, actions/user/day, burst (×3–×5). См. [DAU](#dau), [86_400](#86400). |
| `Storage = rows × row_size × retention` | **Диск на год**: writes/day × 365 × bytes/row. Добавь replication factor если считаешь raw copies. |
| `Replication lag ≤ read staleness SLO` | Replica OK для read только если **lag < допустимый stale** продукта (feed OK, balance — нет). |

→ порядки latency (SSD ~1 ms, cross-region ~100 ms): [workflow/02 §2.1](workflow/02-non-functional-requirements.md#21-цифры-на-доску)

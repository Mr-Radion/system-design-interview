---
layer: product
steps: [6]
related:
  - architecture/messaging-patterns
---

# Message Brokers (брокеры сообщений)

> **Главное:** Message brokers — products for durable messaging. Вход — ordering, retention, throughput.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need partitioned ordered stream? | Да | Kafka |
| Need flexible routing? | Да | RabbitMQ / NATS |

## Цепочка решений

Шаг 5 pattern → шаг 2 Infra product

## Apache Kafka

- ➕ High throughput log; replay; retention; stream processing ecosystem; partition ordering.
- ➖ Heavy ops (ZooKeeper/KRaft); overkill for simple task queue; learning curve.
- 📍 **Where:** event sourcing, activity streams, CDC (Change Data Capture, захват изменений), >100K events/sec.

## RabbitMQ

- ➕ Flexible routing (exchanges); classic task queue; easier ops; AMQP standard.
- ➖ Lower throughput vs Kafka; not a replay log; clustering complexity.
- 📍 **Where:** task queues, RPC (Remote Procedure Call, удалённый вызов процедуры)-style async, moderate volume, complex routing.

## AWS SQS / SNS

- ➕ Fully managed; zero ops; pay per use; SNS fan-out + SQS queue pattern.
- ➖ AWS lock-in; limited ordering (FIFO (First In First Out, первым пришёл — первым вышел) queues separate); no replay log.
- 📍 **Where:** AWS-native, simple async, serverless triggers.

## NATS / JetStream

- ➕ Ultra-low latency; lightweight; simple clustering.
- ➖ Smaller ecosystem vs Kafka.
- 📍 **Where:** IoT, control plane, low-latency messaging.

## Comparison (сравнение)

| Need | Pick |
|------|------|
| Task queue, job workers | RabbitMQ / SQS |
| Event log, replay, streams | Kafka |
| AWS serverless, simple | SQS + SNS |
| Low latency pub/sub | NATS |

## Kafka tuning trade-offs (компромиссы настройки Kafka)

- **More partitions:** higher parallelism, more consumer scale; more overhead.
- **Retention vs Compacted:** retention = replay window; compacted = latest state per key.

---

## Примечания (Habr, часть 4) — Kafka internals

### Когда Kafka (Uber geo example)

Driver location every 2 sec × thousands drivers → direct DB insert = overload.

**Pattern:** Producer → Kafka (high throughput buffer) → Consumer batch-writes to DB every 10 min.

Trade-off: **latency** (location in DB delayed) vs **DB survival** (throughput smoothed).

### Компоненты

| Component | Role |
|-----------|------|
| Producer | Publishes to topic |
| Broker | Kafka server, stores topics |
| Topic | Named channel (`sendEmail`, `writeLocationToDB`) |
| Partition | Parallelism unit within topic |
| Consumer Group | Consumers sharing partition assignment |

**Аналогия:** Broker = DB server, Topic = Table, Partition = Segment.

### Правило partitions ≥ consumers (критично!)

- 1 partition обрабатывается **только 1 consumer** внутри одной consumer group.
- 4 partitions, 3 consumers → 3 busy, 1 idle partition wait.
- **Scale consumers horizontally → создайте ≥ partitions.**

**Разные consumer groups** на одном topic: VideoTranscode group + Subtitles group — каждая обрабатывает все partitions independently (fan-out streams).

### Queue delete vs Stream retain

- **RabbitMQ/SQS:** consumer deletes message after ACK.
- **Kafka:** messages stay (retention policy); consumers advance offset — enables replay + multiple groups.

**Источник:** [часть 4](https://habr.com/ru/articles/893548/)


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

---
layer: pattern
steps: [5]
related:
  - architecture/cap-pacelc-distributed
  - architecture/batch-vs-stream
  - technologies/message-brokers
---

# Distributed Coordination (координация в распределённой системе)

> **Главное:** Distributed coordination — leader election, consensus. Вход — need for consistency and coordination.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need strong consensus? | Да | Consensus (Raft/Paxos) |
| Simple locks only? | Да | Lightweight coordination |

## Цепочка решений

Шаг 2 NFR → шаг 5 coordination → шаг 2 Infra tech

## Leader / Follower (ведущий / ведомый)

**Выбор:** один leader принимает запросы и распределяет work между followers.

- ➕ **Плюсы:** клиент видит «одну систему»; централизованная координация; проще reasoning.
- ➖ **Минусы / Цена:** leader = bottleneck + SPOF (Single Point Of Failure, единая точка отказа) без election; failover latency.
- 📍 **Где применять:** distributed DB internals, Spark/Flink coordinator, custom job schedulers.

## Leader Election as Black Box (выбор лидера как «чёрный ящик»)

**Выбор:** алгоритм (Raft, Bully, Gossip) выбирает leader автоматически.

- ➕ **Плюсы:** автоматический failover; no human intervention; стандартные impl (etcd, ZooKeeper).
- ➖ **Минусы / Цена:** split-brain risk if misconfigured; election storm during instability; operational complexity.
- 📍 **На SD interview:** «представьте election как чёрный ящик» — не кодить Bully, а знать зачем.

## Orchestrator Self-Healing (K8s-like, самовосстановление оркестратора)

**Выбор:** orchestrator следит за workers; при падении — перезапуск; leader orchestrator через election.

- ➕ **Плюсы:** min human ops; SLA (Service Level Agreement, соглашение об уровне сервиса) «≥ N servers always»; auto-recovery.
- ➖ **Минусы / Цена:** orchestrator itself needs HA; cascading restarts; resource thrashing if flapping.
- 📍 **Где применять:** K8s (Kubernetes, оркестратор контейнеров), Nomad, custom fleet manager («≥ 4 servers always» из Habr).

## Big Data: Build vs Use Spark/Flink

**Выбор:** писать свой coordinator/worker vs использовать Spark как black box.

- ➕ **Build custom:** полный контроль.
- ➖ **Build custom:** годы engineering; rebalance, fault recovery, logging — solved problems.
- ➕ **Spark/Flink:** пишете только business logic (Python/Scala job); infra distributed «из коробки».
- ➖ **Spark/Flink:** ops overhead; JVM; overkill для < TB data.
- 📍 **Когда distributed/big data tool:** один инстанс не тянет объём или CPU (ML (Machine Learning, машинное обучение) training, social graph analytics, multi-source ETL (Extract, Transform, Load).

---

## Примечания (Habr, части 5–6)

### Распределённая система — определение

Несколько машин выполняют одну задачу. Примеры из серии:
- **Шардирование** — данные не помещаются на одном сервере.
- **Horizontal scaling** — запросы обрабатывают несколько серверов.
- **Big Data** — coordinator делит dataset на chunks → workers → merge result.

С точки зрения клиента система должна выглядеть как **один компьютер**.

### Leader Election — алгоритмы (для справки, не для кодинга на interview)

| Алгоритм | Сложность | Контекст |
|----------|-----------|----------|
| LCR (Ring) | O(N²) | Учебный |
| HS (Hirschberg-Sinclair) | O(N log N) | Ring |
| Bully (хулиган) | O(N) | Простой |
| Gossip (сплетни) | O(log N) | Cassandra-style |

**Правило interview:** знать *что* при падении leader система re-elects leader, не реализовывать алгоритм.

### Self-healing stack (из части 5)

```
Leader-Orchestrator → следит за Worker-Orchestrators
Worker-Orchestrators → следят за App Servers
Leader упал → Worker становится Leader через election
App Server упал → Worker-Orchestrator перезапускает
```

Аналог: Kubernetes control plane + kubelet, но описано на Habr без K8s (Kubernetes, оркестратор контейнеров) терминов.

### Apache Spark — coordinator/worker

- **Coordinator (Driver):** принимает job, делит data, собирает results.
- **Workers (Executors):** обрабатывают partitions параллельно.
- Coordinator обязан: failover workers, rebalance, merge, logging.

**Когда использовать:** ML (Machine Learning, машинное обучение) training, recommendation systems, multi-source ETL (Extract, Transform, Load) — когда single machine fails.

**Источники:** [часть 5](https://habr.com/ru/articles/900396/), серия Shivam Bhadani / Habr.


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

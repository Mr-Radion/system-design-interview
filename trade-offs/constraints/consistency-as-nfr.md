---
layer: constraint
steps: [2]
related:
  - architecture/cap-pacelc-distributed
  - data/replication-sync-async
---

# Consistency как NFR (согласованность как нефункциональное требование)

> **Главное:** **Первый шаг цепочки CAP.** Вход — бизнес-операция (FR). Выход — strong/eventual/causal → [cap-pacelc](../architecture/cap-pacelc-distributed.md).

## Что определяет выбор

| Операция | Consistency |
|----------|-------------|
| Платёж, инвентарь, бронь | **Strong** |
| Лайки, просмотры | **Eventual** |
| Автор видит свой комментарий | **Causal** |

## Цепочка решений

шаг 2 NFR consistency level → Deep Dive §4.4 CAP + §4.2 replication → HLD §3.1 sync/async → Deep Dive §4.x

## Strong Consistency vs Eventual Consistency (как требование)

На шаге 2 фиксируем **уровень консистентности как бизнес-требование**, не как выбор БД.

### Strong Consistency (строгая)

- ➕ **Плюсы:** пользователь всегда видит актуальные данные; нет «пропавших» денег/заказов.
- ➖ **Минусы / Цена:** выше latency на write; ниже availability при partition (CAP (Consistency, Availability, Partition tolerance — согласованность, доступность, разделение).
- 📍 **Где применять:** баланс счёта, inventory при checkout, бронирование мест (один остаток).

### Eventual Consistency ( eventual)

- ➕ **Плюсы:** высокая availability, низкий latency write, horizontal scale.
- ➖ **Минусы / Цена:** временное расхождение данных между узлами/пользователями.
- 📍 **Где применять:** лайки, просмотры, лента новостей, CDN (Content Delivery Network, сеть доставки контента) cache, social graph.

### Causal Consistency (промежуточный)

- ➕ **Плюсы:** «если A написал комментарий, A его видит» — без global strong.
- ➖ **Минусы / Цена:** сложнее реализация (version vectors).
- 📍 **Где применять:** collaborative editing (Notion-lite), комментарии.

### Чеклист (Checklist) на шаге 2

| Операция | Consistency level |
|----------|-------------------|
| Payment | Strong |
| Like count | Eventual (± few sec OK) |
| Feed ranking | Eventual |
| Seat booking | Strong / linearizable |

**Вопрос из Medium:** «What's OK to delay?» — ответ идёт в эту таблицу.


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

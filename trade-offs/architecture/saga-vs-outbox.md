---
layer: pattern
steps: [5]
related:
  - architecture/orchestration-choreography-saga
  - architecture/messaging-patterns
  - architecture/cap-pacelc-distributed
  - api/write-api-idempotency
  - api/sync-async-messaging
---

# Saga vs Transactional Outbox (сага vs транзакционный outbox)

> **Главное:** это **разные уровни** проблемы.  
> **Outbox** — «записал в БД **и** гарантированно отправил событие» **внутри одного сервиса**.  
> **Saga** — «несколько сервисов участвуют в одной бизнес-транзакции» **без** распределённого 2PC.  
> На практике часто **оба**: Outbox в каждом сервисе + Saga между ними.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Вся логика в **одном** сервисе и одной БД? | Да | **Transactional Outbox** (Saga не нужна) |
| Нужно обновить **2+ сервиса** в одной бизнес-операции? | Да | **Saga** (+ Outbox на публикацию событий) |
| Нужен центральный контроль шагов и компенсаций? | Да | **Orchestration Saga** |
| Простая цепочка «событие → реакция»? | Да | **Choreography Saga** |
| Критично strong consistency **внутри** сервиса? | Да | Outbox в **той же ACID TX** что и business row |
| Webhook / PSP callback может прийти дважды? | Да | **Idempotency** на consumer (дополнение к Outbox/Saga) |

## Частая ошибка на собесе

| Неверно | Верно |
|---------|-------|
| «Outbox заменяет Saga» | Outbox решает **at-least-once publish**; Saga — **multi-service TX** |
| «Saga = Kafka» | Saga — **паттерн**; Kafka — **транспорт**; оркестратор — Temporal/Camunda/custom |
| «2PC между микросервисами» | В production почти всегда **Saga + idempotency**, не XA 2PC |

## Transactional Outbox

**Выбор:** business write и запись в `outbox` — **одна транзакция**; отдельный процесс публикует в брокер.

- ➕ **Плюсы:** не теряем событие при crash после commit; CP для write в БД; простая модель для одного сервиса.
- ➖ **Минусы / Цена:** eventual доставка; poller/relay; дубликаты → нужна idempotency у consumer.
- 📍 **Где:** заказ создан → событие в Kafka; платёж принят в Wallet → уведомить Analytics.

**Tech (шаг 2 Infra):** PostgreSQL `outbox` table + Debezium CDC **или** polling worker; Kafka/RabbitMQ.

→ Детали: [messaging-patterns](messaging-patterns.md), [write-api-idempotency](../api/write-api-idempotency.md).

## Saga (распределённая бизнес-транзакция)

**Выбор:** последовательность локальных TX с **компенсирующими** действиями при сбое.

- ➕ **Плюсы:** согласованность **между** сервисами без 2PC; масштаб по bounded context.
- ➖ **Минусы / Цена:** eventual consistency между шагами; сложный debug; компенсации не всегда симметричны (charge vs refund).
- 📍 **Где:** betting deposit: Wallet reserve → Payment PSP → Wallet confirm; отмена при timeout PSP.

| Стиль | Когда | Минус |
|-------|-------|-------|
| **Orchestration** | Много шагов, compliance, видимость | Single point — orchestrator |
| **Choreography** | 2–3 сервиса, простая цепочка | «Spaghetti» событий при росте |

→ Детали: [orchestration-choreography-saga](orchestration-choreography-saga.md).

## Комбинация (production)

```
Payment Service:  ACID(write payment + outbox) → Kafka PaymentAuthorized
Wallet Service:   consume (idempotent) → ACID(reserve + outbox)
Orchestrator:     saga state machine, timeout → compensate Wallet
```

CAP: **CP** внутри каждого сервиса; **eventual** между сервисами — норма для Saga.

## Резюме

- **Outbox** — один сервис, at-least-once publish после ACID commit.
- **Saga** — несколько сервисов, компенсации, без 2PC.
- Production payments: **оба** + idempotency на consumer.
- 2PC между микросервисами на собесе — red flag.

## FAQ (собес)

| Вопрос | Ответ |
|--------|-------|
| Outbox vs Saga? | Outbox = local TX + event. Saga = multi-service workflow. |
| Outbox vs dual-write? | Dual-write может потерять event; outbox atomic in TX. |
| Orchestration vs choreography? | Orch = central state. Choreo = events only. |
| Компенсация = undo? | Не всегда; adjusting entry, refund, release hold. |
| Kafka = Saga? | Kafka = transport. Saga = pattern. Temporal = orchestrator. |

---

## Сокращения в этом файле

| Аббревиатура | Расшифровка |
|--------------|-------------|
| TX | Transaction — транзакция |
| 2PC | Two-Phase Commit — двухфазный коммит |
| PSP | Payment Service Provider — платёжный провайдер |
| CP | CAP: Consistency + Partition tolerance |

Полный индекс: [GUIDE.md](../../FRAMEWORK.md) · [README.md](../../FRAMEWORK.md#trade-offs).

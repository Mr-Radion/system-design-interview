# Шаг 7 — Technology choices

← [FRAMEWORK](../FRAMEWORK.md)

## Что фиксируем

После §6 HLD — **имена продуктов**, sizing из §2.2, traceability из §5.

## Шаблон §7

### Дерево на каждый класс

| Вопрос | Если да | Если нет |
|--------|---------|----------|
| … | … | … |
| **✅ Выбор** | **…** | … |

### Infra table

| Компонент | Тех | Размер | Откуда |
|-----------|-----|--------|--------|
| … | … | … | §2.2 |

## Чеклист классов

| Класс | Trade-off | Example |
|-------|-----------|---------|
| Broker | [message-brokers](../trade-offs/technologies/message-brokers.md) | Kafka / RabbitMQ |
| Cache | [caches](../trade-offs/technologies/caches.md) | Redis |
| DB | [databases](../trade-offs/technologies/databases.md) | PostgreSQL / Scylla |
| CDN / S3 | [object-storage](../trade-offs/technologies/object-storage.md) | CloudFront + S3 |
| Gateway / LB | [api-gateways](../trade-offs/technologies/api-gateways.md) | ALB / Kong |

Шаблон: [instagram-feed §7](../examples/instagram-feed.md#7-technology-choices)

---

← [06 — HLD](06-high-level-design.md) · [FRAMEWORK](../FRAMEWORK.md)

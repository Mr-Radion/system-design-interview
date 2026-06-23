# Шаг 5 — High-Level Design

← [FRAMEWORK](../FRAMEWORK.md)

## Схема (boxes)

```
Client → Gateway → Service A → DB
                 → Service B → Cache / Queue
```

## Data flow (1 критичный UC)

| # | Компонент | Действие |
|---|-----------|----------|
| 1 | … | … |
| 2 | … | … |

**При сбое:** что деградирует (из trade-offs шага 2).

---

← Назад: [04 — Data](04-data-model.md) · [FRAMEWORK](../FRAMEWORK.md)

Пример: [instagram-feed.md](../examples/instagram-feed.md)

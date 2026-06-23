---
layer: paradigm
steps: [3, 4]
---

# Offset vs Cursor Pagination (пагинация: смещение vs курсор)

> **Главное:** Pagination — cursor vs offset. Вход — query ordering and stability.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Need stable pages with new inserts? | Да | Cursor |
| Simple alphabetical pages and small data? | Да | Offset |

## Цепочка решений

Шаг 2 query semantics → шаг 4 API/data → шаг 2 Infra tech

## Offset (LIMIT/OFFSET)

- ➕ **Плюсы:** trivial; jump to page 150; total count possible.
- ➖ **Минусы / Цена:** `OFFSET 1000000` = scan 1M rows; duplicates/skips if data changes during paging; O(n) degradation.
- 📍 **Где применять:** admin panels, small datasets, SEO pages with page numbers.

## Cursor-based (Keyset)

**Выбор:** `WHERE id > last_seen_id ORDER BY id LIMIT 10`.

- ➕ **Плюсы:** O(1) stable speed at any depth; no duplicates in infinite scroll; consistent under inserts.
- ➖ **Минусы / Цена:** no arbitrary page jump; requires stable unique sort key; bidirectional harder.
- 📍 **Где применять:** Twitter/Instagram feed, chat history, activity logs.

## API (Application Programming Interface, программный интерфейс) contract

```json
// Cursor response
{
  "items": [...],
  "next_cursor": "eyJpZCI6MTIzfQ==",
  "has_more": true
}
```

**Шаг 3:** pagination в API (Application Programming Interface, программный интерфейс) contract. **Шаг 4:** index на sort key (usually PK or `(created_at, id)`).


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

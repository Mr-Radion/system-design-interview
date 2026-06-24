# Шаг 1 — Requirements (5–8 min)

← [FRAMEWORK](../FRAMEWORK.md)

## Что на доске

- **Overview** — одна строка: главная история / pipeline
- **FR-1…FR-N** — **4–6 max** на доске (остальное — Out of scope)
- **Out of scope** — явно что не делаем

**Не на доске в шаге 1:** SLA/SLO (→ шаг 2 NFR), UC→FR таблицы, ER с полями — §3 HLD / §4 Deep Dive по запросу.

## Спроси интервьюера

| Вопрос | Зачем |
|--------|-------|
| Scale? DAU / users? | → шаг 2 NFR |
| Geo? multi-region? | CDN, sharding |
| Read vs write? | bottleneck hint |
| Consistency? money / social? | CAP в §2.6 / Deep Dive §4.4 |

## Шаблон §1

```markdown
**Overview:** post → fan-out → feed cache

| ID | Требование | Пояснение |
|----|------------|-----------|
| **FR-1** | User загружает пост (текст + 1 фото) | Sync ACK metadata; media — presigned upload |
| **FR-2** | Лента подписок — reverse chrono | Pagination; stale OK |
| **FR-3** | Like/unlike идемпотентен | Double-click safe |
| **FR-4** | Celebrity fan-out async | N followers — не sync в POST |

**Out of scope:** DMs, search, geo feed
```

Каждый FR = **что** + **один edge case**, не одна строка.

---

← [FRAMEWORK](../FRAMEWORK.md) · Дальше: [02 — NFR](02-non-functional-requirements.md) →

Примеры: [instagram-feed.md](../examples/instagram-feed.md) · [paypal-payments.md](../examples/paypal-payments.md) · [vk-social.md](../examples/vk-social.md)

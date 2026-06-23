# Шаг 1 — Функциональные требования

← [FRAMEWORK](../FRAMEWORK.md)

## Что фиксируем

- **Overview** — одна строка: главная история / pipeline
- **FR-1…FR-N** — требование + пояснение + edge cases
- **UC → FR** — маппинг use case на FR (не дублировать)
- **Out of scope** — явно что не делаем
- Логическая ER (связи, без типов полей)

## Шаблон §1 (эталон — GM-346 style)

```markdown
**Overview:** post → fan-out → feed cache · bottleneck = read bandwidth

| ID | Требование | Пояснение |
|----|------------|-----------|
| **FR-1** | User загружает пост (текст + 1 фото) | Sync ACK metadata; media — presigned upload, не через API body |
| **FR-3** | Like/unlike идемпотентен | Double-click не создаёт второй like |
| **FR-N** | … | **⚠️ Собес:** offset vs cursor pagination |

### UC → FR

| UC | FR |
|----|-----|
| UC1 Загрузить пост | FR-1, FR-7, FR-8 |
| UC2 Лента | FR-2, FR-5, FR-6 |

**Out of scope:** DMs, search, geo feed

**ER:** User 1──M Post · User M──N User
```

**10–12 FR** на capstone-пример; **8+** на простой. Каждый FR = **что** + **почему/edge case**, не одна строка.

## Акторы

| Актор | Роль |
|-------|------|
| Пользователь | *кто инициирует действие* |
| Client / UI | *приложение, формы, вызовы API* |
| Backend | *сервис домена* |
| Cron / Scheduler | *ночные job, очистка, retry, отчёты* |
| *внешняя система* | *платежи, storage, …* |

## Реестр UC

| UC | Название | Приоритет | FR |
|----|----------|-----------|-----|
| UC1 | *Глагол + Объект* | Must | FR-1, … |
| UC2 | … | Must | FR-2, … |
| UC3 | … | Should | — |

Формулировка UC: **«Инфинитив + Объект»** (Просмотреть список, Загрузить файл).

## Интеграции

| Система | Зачем |
|---------|-------|
| … | … |

---

← [FRAMEWORK](../FRAMEWORK.md) · Дальше: [02 — NFR](02-non-functional-requirements.md) →

Примеры: [instagram-feed.md](../examples/instagram-feed.md) · [paypal-payments.md](../examples/paypal-payments.md) · [vk-social.md](../examples/vk-social.md)

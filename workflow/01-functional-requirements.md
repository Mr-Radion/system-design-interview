# Шаг 1 — Функциональные требования

← [FRAMEWORK.md](../FRAMEWORK.md)

**Пример:** лента постов (Instagram-like)

## Фичи

- Загрузить пост (текст + фото)
- Получить ленту (обратный хронологический порядок, подписки)
- Лайк, комментарий
- Подписка / отписка

## Акторы

User · Mobile/Web Client · Feed API · Media Service · Social Graph Service

## Use Cases

| UC | Действие |
|----|----------|
| UC1 | Опубликовать пост |
| UC2 | Открыть ленту |
| UC3 | Лайк / комментарий |
| UC4 | Подписаться / отписаться |

## Интеграции

CDN + Object Storage (фото) · Push-уведомления (опционально)

## Логическая ER

```
User 1──M Post
User M──N User (follow)
Post 1──M Like, Comment
```

---

← [FRAMEWORK](../FRAMEWORK.md) · Дальше: [02 — NFR](02-non-functional-requirements.md) →

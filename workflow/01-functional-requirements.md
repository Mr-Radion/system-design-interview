# Шаг 1 — Функциональные требования (5–8 min)

← [FRAMEWORK](../FRAMEWORK.md)

**Фокус шага:** *что* делает система — без цифр, SLA и trade-offs (→ шаг 2 NFR).

**На доске:** Overview + **4–6 core FR** + Out of scope.  
**Не на доске:** RPS, SLA (→ шаг 2) · trade-off решения и вендоры (→ шаг 4). На HLD (шаг 3) — роли компонентов OK, см. [03 §3](03-high-level-design.md).

---

## Что обычно входит в FR — спроси и зафиксируй

### 1. Поведение при запросах (API, бизнес-правила)

| Спроси | Пример |
|--------|--------|
| Что делает каждый core endpoint? Sync или async ACK? | POST пост → sync ACK metadata; фото — presigned upload |
| Какие бизнес-правила на операции? | Like идемпотентен; unfollow сразу убирает из ленты |
| Что в ответе / ошибках? | Insufficient funds → fail без partial debit |

→ таблица **FR-1…FR-N** (требование + краткое пояснение).

### 2. Потоки взаимодействия (use cases)

| Спроси | Пример |
|--------|--------|
| Главные сценарии end-to-end? | Загрузить пост → fan-out → лента |
| Приоритет Must / Should? | UC1 загрузка — Must; UC6 stories — Should → out of scope |

→ **UC → FR** · UC = **«Инфинитив + Объект»** (Просмотреть ленту, Загрузить пост):

`| UC1 Загрузить пост | Must | FR-1 |` · `| UC2 Открыть ленту | Must | FR-2, FR-6 |`

### 3. Роли, ограничения, доступ

| Спроси | Пример |
|--------|--------|
| Кто инициирует действия? (**акторы** — не дублируют UC) | User · Mobile Client · Feed API · Media Service |
| Роли / права? | Только автор удаляет свой пост; admin — pull |
| Лимиты продукта? | Clan ≤ 50 members; pagination обязательна |

**Актор** = *участник системы*. **UC** = *сценарий*. Один актор — много UC.

### 4. Интеграции с внешними сервисами

| Спроси | Пример |
|--------|--------|
| Какие внешние системы? | Object storage (media) · Payment (IAP) · OAuth (login) |
| Кто инициатор, sync/async? | Billing webhook → async grant currency |

→ при необходимости короткая таблица: `| Object storage | media upload | FR-1 |`

---

## Что фиксируем (структура §1)

| Блок | Содержание |
|------|------------|
| **Overview** | одна строка: pipeline / главная история |
| **FR-1…FR-N** | из п.1 — что + пояснение (правило или ограничение) |
| **UC → FR** | из п.2 — одна строка на UC |
| **Акторы** | из п.3 — кто участвует (без повтора UC) |
| **Интеграции** | из п.4 — если есть внешние системы |
| **Out of scope** | явно что **не** делаем |
| **Логическая ER** | связи сущностей, без типов полей → детали §3.2 HLD |

---

## Шаблон §1

```markdown
**Overview:** post → async fan-out → feed cache

| ID | Требование | Пояснение |
|----|------------|-----------|
| **FR-1** | User загружает пост (текст + 1 фото) | Sync ACK; media — presigned upload |
| **FR-2** | Лента подписок — reverse chrono | Pagination |
| **FR-3** | Like/unlike идемпотентен | Повторный click — тот же результат |
| **FR-4** | Follow/unfollow | Unfollow сразу убирает из ленты |

**UC → FR:** UC1 Загрузить пост → FR-1 · UC2 Лента → FR-2 · UC3 Like → FR-3

**Акторы:** User · Client · Feed API · Media Service

**Интеграции:** Object storage — media (FR-1)

**Out of scope:** DMs, search, geo feed

**ER:** User 1──M Post · User M──N User · Post 1──M Like
```

Полный список FR в заметках — **8+**; на доске — **4–6 core**.

---

## Перед шагом 2

- [ ] Overview · core FR · Out of scope
- [ ] Цифры не писали → [02 — NFR](02-non-functional-requirements.md), сначала **§2.0 спроси интервьюера**

---

← [FRAMEWORK](../FRAMEWORK.md) · [02 — NFR](02-non-functional-requirements.md) →

Примеры: [instagram](../examples/instagram-feed.md) · [paypal](../examples/paypal-payments.md) · [vk](../examples/vk-social.md) · [open-world](../examples/open-world-mobile-game.md) · [nutrition](../examples/nutrition-mobile-app.md) · [bulk-messaging](../examples/bulk-messaging-platform.md)

# Шаг 1 — Функциональные требования (5–8 min)

← [FRAMEWORK](../FRAMEWORK.md)

**Фокус шага:** *что* делает система — без цифр, SLA и trade-offs (→ шаг 2 NFR).

## Что фиксируем

- **Overview** — одна строка: главная история / pipeline
- **FR-1…FR-N** — требование + пояснение + edge cases
- **UC → FR** — маппинг use case на FR (не дублировать текст)
- **Out of scope** — явно что не делаем
- **Акторы** — кто участвует в UC
- **Реестр UC** — Must / Should, приоритет
- **Интеграции** — внешние системы (без имён вендоров, если не дали)
- **Логическая ER** — связи сущностей, без типов полей

**На доске в 5–8 min:** Overview + **4–6 core FR** + Out of scope.  
**В заметках / если спросят:** полный список FR, UC→FR, акторы, ER (детали ER → §3.2 HLD).

**Не на доске в шаге 1:** SLA/SLO, RPS, RPO (→ шаг 2), имена Kafka/Redis (→ шаг 4).

## Алгоритм (5–8 min)

1. Переформулируй задачу → **Overview** (1 строка)
2. Выдели **3–5 core UC** → реестр UC
3. Из UC выпиши **FR** с edge case в пояснении
4. Явно **Out of scope** — сужает дизайн
5. Акторы + интеграции — если влияют на FR
6. ER — только связи, если успеваешь или спросят

## Шаблон §1 (эталон)

```markdown
**Overview:** post → async fan-out → feed cache

| ID | Требование | Пояснение |
|----|------------|-----------|
| **FR-1** | User загружает пост (текст + 1 фото) | Sync ACK metadata; media — presigned upload, не через API body |
| **FR-2** | Лента подписок — reverse chrono | Pagination; stale OK (секунды) |
| **FR-3** | Like/unlike идемпотентен | Double-click не создаёт второй like |
| **FR-4** | Follow/unfollow — strong consistency | Unfollow сразу убирает из ленты |
| **FR-5** | Celebrity fan-out async | N followers — не sync в POST |
| **FR-6** | Read >> write по bandwidth | Драйвер для NFR §2.2 |

### UC → FR

| UC | FR |
|----|-----|
| UC1 Загрузить пост | FR-1 |
| UC2 Открыть ленту | FR-2, FR-6 |
| UC3 Like/unlike | FR-3 |
| UC4 Follow/unfollow | FR-4 |
| UC5 Celebrity публикует | FR-5 |

**Out of scope:** DMs, search, geo feed, video transcode

**ER:** User 1──M Post · User M──N User · Post 1──M Like
```

**Объём FR:** capstone **8–12** в заметках · на доске **4–6 core**.  
Каждый FR = **что** + **почему / edge case**, не одна строка.

## Акторы

| Актор | Роль |
|-------|------|
| Пользователь | *кто инициирует действие* |
| Client / UI | *приложение, формы, вызовы API* |
| Backend | *сервис домена* |
| Cron / Scheduler | *ночные job, очистка, retry, отчёты* |
| *внешняя система* | *платежи, storage, OAuth, …* |

## Реестр UC

| UC | Название | Приоритет | FR |
|----|----------|-----------|-----|
| UC1 | *Глагол + Объект* | Must | FR-1, … |
| UC2 | … | Must | FR-2, … |
| UC3 | … | Should | — |

Формулировка UC: **«Инфинитив + Объект»** (Просмотреть ленту, Загрузить пост).

## Интеграции

| Система | Зачем | FR |
|---------|-------|-----|
| Object storage | media upload | FR-1 |
| Payment provider | IAP / checkout | FR-… |
| OAuth provider | social login | FR-… |

## Логическая ER

```
Сущность A 1──M Сущность B
Сущность B M──N Сущность C
```

Связи здесь; типы полей и индексы → §3.2 HLD / §4.2 по запросу.

## Чеклист перед шагом 2

- [ ] Overview одной строкой
- [ ] Core FR с edge cases
- [ ] Out of scope явно
- [ ] UC→FR согласован
- [ ] Цифры **не** писали — переходи в [02 — NFR](02-non-functional-requirements.md) и **сначала спроси интервьюера**

---

← [FRAMEWORK](../FRAMEWORK.md) · Дальше: [02 — NFR](02-non-functional-requirements.md) →

Примеры: [instagram-feed.md](../examples/instagram-feed.md) · [paypal-payments.md](../examples/paypal-payments.md) · [vk-social.md](../examples/vk-social.md) · [open-world-mobile-game.md](../examples/open-world-mobile-game.md)

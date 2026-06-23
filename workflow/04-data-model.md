# Шаг 4 — Data Model

← [FRAMEWORK.md](../FRAMEWORK.md)

**Пример:** PostgreSQL (профиль, граф) + Redis (лента) + S3 (медиа)

| Таблица | Ключ | Заметка |
|---------|------|---------|
| `users` | `user_id` UUID | email, password_hash |
| `posts` | `post_id` UUID | `user_id`, text, `media_url`, `created_at` |
| `follows` | `(follower_id, followee_id)` | индекс по `follower_id` |
| `likes` | `(user_id, post_id)` | unique |
| `comments` | `comment_id` | `post_id`, text |

**Redis:** `feed:{user_id}` → List of `post_id` (top 200)

**Индексы:** `posts(user_id, created_at DESC)`, `follows(follower_id)`

**Consistency:** posts/likes — write primary; feed read — Redis (eventual); follows read — replica OK.

---

← Назад: [03 — API](03-api-design.md) · [FRAMEWORK](../FRAMEWORK.md) · Дальше: [05 — HLD](05-high-level-design.md) →

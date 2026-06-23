# Framework System Design

Фреймворк собеса на примере **ленты постов (Instagram-like)**.

**Сценарий:** 50M users · p99 ≤ 2s · 10 постов/запрос · 1 фото/пост

## Workflow

| Шаг | Файл |
|-----|------|
| 1. Функциональные требования | [workflow/01-functional-requirements.md](workflow/01-functional-requirements.md) |
| 2. NFR + trade-offs (с расчётами) | [workflow/02-non-functional-requirements.md](workflow/02-non-functional-requirements.md) |
| 3. API | [workflow/03-api-design.md](workflow/03-api-design.md) |
| 4. Data model | [workflow/04-data-model.md](workflow/04-data-model.md) |
| 5. High-Level Design | [workflow/05-high-level-design.md](workflow/05-high-level-design.md) |

Формат шага 2: сворачиваемые секции · **Трейдоф** (A / B) · **Выбор**.

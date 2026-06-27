# system-design-interview

Шпаргалка System Design для собесов — **формат Alex Xu, 45 мин**.

**Старт:** [FRAMEWORK.md](FRAMEWORK.md)

## Workflow (45 min)

| Шаг | Время | Файл |
|-----|-------|------|
| 1. Requirements | 5–8 min | [Функциональные требования](workflow/01-functional-requirements.md) |
| 2. NFR | 5–7 min на доске | [§2.1 цифры · §2.2 pillars + вывод](workflow/02-non-functional-requirements.md) |
| 3. High-level design | 12–15 min | [API · schema · схема](workflow/03-high-level-design.md) |
| 4. Deep Dive + Tech | 15–18 min | [По выводу §2.2 NFR](workflow/04-deep-dive.md) |

## Примеры

| Пример | Фокус |
|--------|-------|
| [Instagram-like feed](examples/instagram-feed.md) | read-heavy · CDN · cache |
| [PayPal-like payments](examples/paypal-payments.md) | saga · outbox · CP ledger |
| [VK-like social](examples/vk-social.md) | messaging · social graph |
| [Mobile open-world game](examples/open-world-mobile-game.md) | 20K CCU · progress · telemetry · billing |
| [Mobile nutrition / calorie app](examples/nutrition-mobile-app.md) | AI scan · IAP billing · events pipeline |
| [B2B bulk messaging platform](examples/bulk-messaging-platform.md) | Kafka dispatch · DLR · ClickHouse analytics |

## Trade-offs

47 тем + [GLOSSARY.md](GLOSSARY.md) — индекс в [FRAMEWORK.md](FRAMEWORK.md#trade-offs)

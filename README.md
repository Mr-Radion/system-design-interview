# system-design-interview

Шпаргалка System Design для собесов.

**Старт:** [FRAMEWORK.md](FRAMEWORK.md)

## Workflow

1. [Функциональные требования](workflow/01-functional-requirements.md)
2. [NFR — цифры и SLO](workflow/02-non-functional-requirements.md)
3. [API Design](workflow/03-api-design.md)
4. [Data Model](workflow/04-data-model.md)
5. [Архитектурные характеристики](workflow/05-architectural-characteristics.md)
6. [High-Level Design](workflow/06-high-level-design.md)
7. [Technology choices](workflow/07-technology-choices.md) — [examples §7](examples/instagram-feed.md#7-technology-choices)

## Примеры

| Пример | Фокус |
|--------|-------|
| [Instagram-like feed](examples/instagram-feed.md) | read-heavy · CDN · sharding · cache |
| [PayPal-like payments](examples/paypal-payments.md) | saga · outbox · idempotency · ledger CP |
| [VK-like social](examples/vk-social.md) | capstone · messaging · social graph |

## Trade-offs

47 тем + [GLOSSARY.md](GLOSSARY.md) — индекс в [FRAMEWORK.md](FRAMEWORK.md#trade-offs)

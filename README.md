# system-design-interview

Шпаргалка System Design для собесов.

**Старт:** [FRAMEWORK.md](FRAMEWORK.md)

## Workflow

1. [Функциональные требования](workflow/01-functional-requirements.md)
2. [NFR + trade-offs](workflow/02-non-functional-requirements.md)
3. [API Design](workflow/03-api-design.md)
4. [Data Model](workflow/04-data-model.md)
5. [High-Level Design](workflow/05-high-level-design.md)  
6. Technology choices — в [examples](examples/) §6

## Примеры

| Пример | Фокус |
|--------|-------|
| [Instagram-like feed](examples/instagram-feed.md) | read-heavy · CDN · sharding · cache |
| [PayPal-like payments](examples/paypal-payments.md) | saga · outbox · idempotency · ledger CP |
| [VK-like social](examples/vk-social.md) | capstone · messaging · social graph |

## Trade-offs

47 тем + [GLOSSARY.md](GLOSSARY.md) — индекс в [FRAMEWORK.md](FRAMEWORK.md#trade-offs)

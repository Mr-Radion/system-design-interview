---
layer: pattern
steps: [5]
related:
  - technologies/object-storage
---

# CDN (Content Delivery Network, сеть доставки контента) & Object Storage (CDN (Content Delivery Network, сеть доставки контента) и объектное хранилище)

> **Главное:** CDN / Object storage — where to place static & large objects. Вход — delivery latency and cost.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Large static assets? | Да | CDN + object storage |
| Need origin processing? | Да | edge computing |

## Цепочка решений

шаг 2 NFR → Deep Dive §4.x → Deep Dive §4.x (tech)

## CDN (Content Delivery Network, сеть доставки контента) vs Origin-only

**CDN (Content Delivery Network, сеть доставки контента):** edge caches static/dynamic content near users.

- ➕ **Плюсы:** low latency globally; origin offload 80-95%; DDoS absorption.
- ➖ **Минусы / Цена:** cache invalidation; cost; stale content at edge; complexity.
- 📍 **Где применять:** images, video, JS/CSS, API (Application Programming Interface, программный интерфейс) responses with TTL (Time To Live, время жизни).

## Push CDN (Content Delivery Network, сеть доставки контента) vs Pull CDN

| | Push | Pull |
|--|------|------|
| Content | Pre-upload to edge | Fetch on first request |
| Use | Large static releases | User-generated content |

## Object Storage (BLOB (Binary Large Object, большой двоичный объект) Pattern

**Выбор:** metadata in DB, binary in object store (S3).

- ➕ **Плюсы:** cheap storage; unlimited scale; CDN (Content Delivery Network, сеть доставки контента) integration; multipart upload.
- ➖ **Минусы / Цена:** no partial update; eventual listing consistency; latency vs block storage.
- 📍 **Где применять:** photos, videos, backups, data lake.

## Storage Class Tiers

| Tier | Access | Cost |
|------|--------|------|
| Hot (Standard) | Frequent | $$$ |
| Warm (IA) | Monthly | $$ |
| Cold (Glacier) | Archive | $ |

## Presigned URL vs Proxy-through Backend

| | Presigned | Proxy |
|--|-----------|-------|
| Load on app | None | High |
| Control | TTL (Time To Live, время жизни), permissions | Full |
| Use | Direct upload/download | Transform on fly |

см. [Infra-таблицу шаг 2 NFR](../../workflow/02-non-functional-requirements.md).

---

## Примечания (Habr, часть 4)

### BLOB (Binary Large Object, большой двоичный объект) — почему не в MySQL/MongoDB

MP4/PNG/PDF as binary (zeros and ones). 1GB video in relational DB → slow queries + you manage scale/backup/availability yourself.

**Pattern:** metadata (filename, url, user_id) in DB → file bytes in S3/R2.

### CDN (Content Delivery Network, сеть доставки контента) flow

1. User request → nearest **edge server** (GeoDNS).
2. **Cache hit** → return from edge.
3. **Cache miss** → fetch from **origin** (S3) → cache at edge + return.
4. Next requests → edge hit.

**Concepts:** edge server, origin server, TTL (e.g. 24h for images), GeoDNS routing.

**Examples:** CloudFront, Cloudflare.

### S3 highlights (product notes in examples §4)

11 nines durability, IAM (Identity and Access Management, управление доступом) policies, presigned URLs, pay-as-you-go — cheaper per GB than RDS for files.

**Источник:** [часть 4](https://habr.com/ru/articles/893548/)


---

## Сокращения в этом файле

| Аббревиатура | Расшифровка |
|--------------|-------------|
| FR | Functional Requirements — функциональные требования |
| NFR | Non-Functional Requirements — нефункциональные требования |
| HLD | High-Level Design — высокоуровневый дизайн |
| RPS | Requests Per Second — запросов в секунду |
| CAP | Consistency, Availability, Partition tolerance |
| LB | Load Balancer — балансировщик нагрузки |

Полный индекс: [README.md](../../FRAMEWORK.md#trade-offs).

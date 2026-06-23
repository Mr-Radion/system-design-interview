---
layer: product
steps: [6]
related:
  - architecture/cdn-object-storage-pattern
---

# Object Storage (объектное хранилище)

> **Главное:** Object storage — large binary storage. Вход — cost and access patterns.

## Что определяет выбор

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Many large objects? | Да | Object storage (S3) |
| Need CDN fronting? | Да | CDN + origin |

## Цепочка решений

Шаг 5 pattern → шаг 2 Infra product

## AWS S3

- ➕ Industry standard; 11 nines durability; lifecycle policies; CDN (Content Delivery Network, сеть доставки контента) integration; presigned URLs.
- ➖ AWS lock-in; egress costs; eventual consistency on LIST (legacy concern reduced).
- 📍 **Default** for cloud object storage.

## Google Cloud Storage (GCS)

- ➕ Strong GCP integration; similar API (Application Programming Interface, программный интерфейс) to S3; good for multi-cloud with abstraction.
- ➖ Smaller third-party ecosystem vs S3.
- 📍 **Where:** GCP-native stacks.

## MinIO

- ➕ S3-compatible; self-hosted; on-prem/K8s (Kubernetes, оркестратор контейнеров); no vendor lock-in.
- ➖ You manage durability, scaling, ops.
- 📍 **Where:** on-prem, air-gapped, hybrid cloud.

## Block (EBS) vs Object (S3) vs File (NFS) (блочное vs объектное vs файловое хранилище)

| Type | Use | Don't use for |
|------|-----|---------------|
| Object (S3) | Photos, videos, backups | Database files |
| Block (EBS) | DB disk, VM root | Direct browser upload |
| File (EFS/NFS) | Shared filesystem | Web-scale static assets |

## Storage Classes (S3) (классы хранения)

| Class | Cost | Retrieval |
|-------|------|-----------|
| Standard | $$$ | Instant |
| IA | $$ | Instant, min storage duration |
| Glacier | $ | Minutes-hours |

**Cost trade-off:** 90% archive → Glacier saves 80%+ vs Standard.


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

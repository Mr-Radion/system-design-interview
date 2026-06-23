# Пример: Instagram-like feed

← [FRAMEWORK.md](../FRAMEWORK.md)

**Сценарий:** 50M users · p99 ≤ 2s · 10 постов/запрос · 1 фото ~200 KB

---

## 1. FR

### Акторы

| Актор | Роль |
|-------|------|
| User | Посты, лента |
| Client | Mobile/Web UI |
| Feed Service | Лента, cache |
| Post Service | Посты |
| Media Service | Upload S3 |
| Social Service | Follow |
| **Cron** | Evict stale feed keys, retry failed fan-out, nightly stats |

### Реестр UC

| UC | Название | Приоритет | Зависимость |
|----|----------|-----------|-------------|
| UC1 | Опубликовать пост | Must | — |
| UC2 | Открыть ленту | Must | UC4 |
| UC3 | Лайк / комментарий | Must | UC1 |
| UC4 | Подписаться / отписаться | Must | — |

### Логическая ER

```
User 1──M Post · User M──N User (follow) · Post 1──M Like, Comment
```

---

## 2. NFR + trade-offs

### Performance

#### Latency SLA

Feed p99 ≤ 2s. Фото — **не** в backend latency (CDN).

> **✅ Выбор:** **p99** + push-модель ленты.

#### Read RPS

```
50M DAU × 20 feed/day ÷ 86_400 ≈ 11_600 baseline · peak ×3 ≈ 35_000 RPS
```

> **✅ Выбор:** гибрид push (&lt; 10K подписчиков) + pull для celebrity.

#### Write RPS

```
50M × 0.05 пост/день ÷ 86_400 ≈ 29 · пик ×5 ≈ 145 RPS
```

> **✅ Выбор:** Kafka async fan-out.

#### Capacity — трафик

```
API read egress  = 35_000 × 4 KB ≈ 140 MB/s  (метаданные ленты)
API write egress = 145 × 1 KB ≈ 0.15 MB/s
Media egress     = 35_000 × 10 × 200 KB ≈ 70 GB/s peak  → только CDN, не origin
```

> **Трейдоф**
>
> **Вариант A — S3 direct:** 350K img req/s к origin — throttling.
>
> **Вариант B — CDN (Cloudflare):** edge cache, p99 фото &lt; 50ms.
>
> **✅ Выбор:** **CDN обязателен** — это performance + capacity, не «просто infra».

#### CDN

См. расчёт media egress выше. Hit rate target ≥ 95%.

> **✅ Выбор:** **Cloudflare** (или CloudFront) перед S3.

### Scalability

Redis feed ~80 GB (50M × 1.6 KB). DB vertical + replicas сначала.

> **✅ Выбор:** write-through cache active users; sharding PG — позже.

### Consistency · Reliability · Security

| Тема | Выбор |
|------|-------|
| CAP | AP feed · CP profile |
| Idempotency | `X-Idempotency-Key` |
| SLA | 99.9% |
| Auth | JWT + blacklist |
| Rate limit | 100 read / 10 write per user |

### Infra — итог (из расчётов)

| Компонент | Технология | Размер / конфиг | Откуда |
|-----------|------------|-----------------|--------|
| CDN | **Cloudflare** Pro/Business | ~70 GB/s peak media | Capacity media |
| Object storage | **S3 Standard** | ~500 TB/год новых фото¹ | 2.5M пост/день × 200 KB |
| Message broker | **Kafka**, 3 brokers | ~50 partitions, RF=3 | fan-out 145 w/s |
| Cache | **Redis** cluster | **~100 GB RAM** (80 GB data + 25%) | feed lists |
| DB | **PostgreSQL** 15 | **~500 GB** disk², 1 primary + 2 replica | OLTP metadata |
| Feed workers | **K8s** | ~20 pods (fan-out CPU) | async consumer lag |
| API | **K8s** | ~30 pods × 2 vCPU | 35K RPS ÷ ~1.2K RPS/pod |
| Gateway | **Envoy** / ALB L7 | rate limit, JWT | — |

¹ Медиа в S3; lifecycle → Glacier для старого.  
² Посты + users + graph; без raw media.

---

## 3. API

| Метод | Назначение |
|-------|------------|
| `POST /posts` | Создать пост |
| `POST /media/upload` | Presigned S3 URL |
| `GET /feed?cursor=` | Лента |
| `POST /posts/{id}/like` | Лайк |
| `POST /users/{id}/follow` | Подписка |

---

## 4. Data

PostgreSQL · Redis `feed:{user_id}` · S3 private + Cloudflare signed URL.

---

## 5. HLD

```
Client → Cloudflare → Envoy → Feed/Post/Media/Social
Kafka ← publish ← Feed Workers → Redis fan-out
S3 ← media upload
PostgreSQL ← metadata
Cron → cleanup / retry jobs
```

---

← [FRAMEWORK.md](../FRAMEWORK.md)

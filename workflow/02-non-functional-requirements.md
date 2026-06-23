# Шаг 2 — NFR + trade-offs

← [FRAMEWORK](../FRAMEWORK.md)

## Что фиксируем

Цифры с собеса + **трейдоф** (A / B → **Выбор**). В конце — **таблица Infra** по результатам расчётов.

## Расчёты RPS

```
Read RPS  = DAU × reads/day ÷ 86_400 × peak
Write RPS = DAU × writes/day ÷ 86_400 × peak
```

## Расчёты Capacity (трафик, storage)

```
API egress     = RPS_peak × размер_ответа_JSON
Media egress   = read_RPS_peak × постов_в_ответе × размер_фото   → через CDN
Storage OLTP   = write_RPS × размер_строки × 31_536_000 × 1.2
Storage media  = постов_в_день × размер_файла × 365
RAM cache      = активных_ключей × размер_значения (+ запас 30%)
```

## Чеклист тем

| Блок | Темы |
|------|------|
| **Performance** | Latency SLA · Read/Write RPS · **Capacity (трафик)** · **CDN** |
| Scalability | Vertical vs horizontal · cache |
| Consistency | CAP · strong · idempotency |
| Reliability | SLA · circuit breaker · backup |
| Security | Auth · rate limit · signed URL |
| **Infra (итог)** | Таблица: технология + **RAM / disk / кластер** из расчётов выше |

**CDN** — в Performance (latency, bandwidth). **Infra** — конкретные продукты и размеры.

## Формат одной темы

> **Трейдоф** → **Вариант A** / **Вариант B** → **✅ Выбор**

## Infra — итоговая таблица (в конце шага 2)

| Компонент | Технология | Размер / конфиг | Откуда цифра |
|-----------|------------|-----------------|--------------|
| CDN | Cloudflare / CloudFront | … Gbps peak | Capacity media |
| Object storage | S3 Standard / IA | … TB | Storage media |
| Message broker | Kafka, N brokers | … partitions | Write RPS + fan-out |
| Cache | Redis cluster | … GB RAM | RAM cache |
| DB | PostgreSQL | … GB disk, M replicas | Storage OLTP |
| API / workers | K8s / ECS | N pods, vCPU | RPS ÷ RPS на pod |
| Gateway | Envoy / ALB | L7 | — |

---

← Назад: [01 — FR](01-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · Дальше: [03 — API](03-api-design.md) →

Пример с цифрами: [instagram-feed.md](../examples/instagram-feed.md)

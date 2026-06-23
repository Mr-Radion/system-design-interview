# Шаг 2 — NFR + trade-offs

← [FRAMEWORK.md](../FRAMEWORK.md)

**Сценарий:** 50M users · p99 ≤ 2s · 10 постов/запрос · 1 фото/пост

Секции сворачиваются. В каждой: контекст → **Трейдоф** (A / B) → **Выбор**.

---

<details open>
<summary><strong>Performance</strong> — Latency &amp; Throughput</summary>

<details open>
<summary>🕒 Latency SLA</summary>

Feed p99 ≤ 2s от запроса до first meaningful paint. Один пост p99 ≤ 1s. Фото через CDN — **не** в backend latency.

> **Трейдоф**
>
> **Вариант A — Strict p99 ≤ 2s:** Redis feed cache + CDN. Дороже — hot cache на пользователя.
>
> **Вариант B — Только p50 ≤ 2s:** проще, но 1% users (500k) — плохой UX.
>
> **✅ Выбор:** **p99** + push-модель ленты — единственный путь на 50M DAU.

</details>

<details>
<summary>📊 Throughput — Read RPS</summary>

```
50M DAU × 20 feed/day ÷ 86_400 ≈ 11_600 RPS baseline
Peak ×3 ≈ 35_000 RPS · 10 постов + 10 URL фото на запрос
```

> **Трейдоф**
>
> **Вариант A — Pull (fan-in on read):** лента из БД на запрос. При 35k RPS DB не выдержит.
>
> **Вариант B — Push (fan-out on write):** Redis List на подписчика. Read O(1), write дорогой у celebrity.
>
> **✅ Выбор:** **Гибрид** — push &lt; 10K подписчиков, pull+merge для celebrity.

</details>

<details>
<summary>📤 Throughput — Write RPS</summary>

```
50M × 0.05 постов/день ÷ 86_400 ≈ 29 RPS · пик ×5 ≈ 145 RPS
Каждый пост → fan-out в N очередей
```

> **Трейдоф**
>
> **Вариант A — Sync fan-out** в HTTP-запросе — timeout у celebrity.
>
> **Вариант B — Kafka + worker** — автор не ждёт, доставка ~1–3s.
>
> **✅ Выбор:** **Kafka + async fan-out**, SLA доставки ≤ 5s.

</details>

</details>

---

<details>
<summary><strong>Масштабируемость</strong></summary>

<details>
<summary>🗄 Horizontal vs Vertical</summary>

API/workers stateless + autoscale. DB vertical + read-replicas. Redis sharded.

> **Трейдоф**
>
> **Вариант A — Vertical + replicas** — проще, хватает при cache.
>
> **Вариант B — Sharding с нуля** — ops ×5, боль с join.
>
> **✅ Выбор:** **Vertical + replicas** сначала. Sharding по `user_id` — когда primary не тянет write.

</details>

<details>
<summary>⚡ Кэш ленты (Redis)</summary>

List `user_id → [post_id…]`, TTL 48h, 200 id · ~80 GB на 50M users.

> **Трейдоф**
>
> **Вариант A — Cache-aside** — cold start.
>
> **Вариант B — Write-through** — всегда warm, дороже write.
>
> **✅ Выбор:** **Гибрид** — write-through active, cache-aside после TTL evict.

</details>

</details>

---

<details>
<summary><strong>Infra</strong></summary>

<details>
<summary>🌐 CDN</summary>

Фото ~200 KB, upload presigned S3, read CloudFront, hit ≥ 95%.

> **Трейдоф**
>
> **Вариант A — S3 direct** — 350K req/s к S3 при 35k RPS.
>
> **Вариант B — CDN** — edge, p99 &lt; 50ms.
>
> **✅ Выбор:** **CDN обязателен.**

</details>

<details>
<summary>↔️ Load Balancer L7</summary>

Envoy/nginx, API Gateway: auth, rate limit, routing.

> **Трейдоф**
>
> **Вариант A — L4** — нет path routing.
>
> **Вариант B — L7** — `/feed`, JWT, rate limit.
>
> **✅ Выбор:** **L7.** L4 только перед L7.

</details>

</details>

---

<details>
<summary><strong>Консистентность</strong></summary>

<details>
<summary>🔀 CAP AP vs CP</summary>

Лента — AP, eventual 1–5s.

> **Трейдоф**
>
> **Вариант A — CP** — availability падает при partition.
>
> **Вариант B — AP** — норма для social.
>
> **✅ Выбор:** **AP** для feed. **CP** — profile/money.

</details>

<details>
<summary>🔒 Strong consistency</summary>

Profile, password — primary. Follows — write primary, read replica ~100ms lag OK.

> **✅ Выбор:** Replica для follows. Primary для password.

</details>

<details>
<summary>🔄 Idempotency</summary>

`X-Idempotency-Key`, Redis 24h.

> **✅ Выбор:** Обязательно на все write API.

</details>

</details>

---

<details>
<summary><strong>Надёжность</strong></summary>

<details>
<summary>❤️ SLA 99.9%</summary>

> **Вариант A — 99.9%** multi-AZ.
>
> **Вариант B — 99.99%** multi-region ×3–5 cost.
>
> **✅ Выбор:** **99.9%** на старте.

</details>

<details>
<summary>🛡 Circuit breaker</summary>

>50% errors / 10s → stale feed 5 min из cache.

> **✅ Выбор:** Breaker + stale fallback.

</details>

<details>
<summary>💾 Backup PITR</summary>

> **✅ Выбор:** PITR + WAL, RPO ≤ 5 min.

</details>

</details>

---

<details>
<summary><strong>Безопасность</strong></summary>

<details>
<summary>🔑 JWT</summary>

Access 15m + refresh 30d httpOnly. Stateless на gateway.

> **✅ Выбор:** JWT + blacklist для logout/ban.

</details>

<details>
<summary>⏱ Rate limit</summary>

100 RPS/user read, 10 write, burst 200/10s.

> **✅ Выбор:** Token bucket Redis (Lua).

</details>

<details>
<summary>🖼 Медиа</summary>

S3 private + signed URL TTL 1h.

> **✅ Выбор:** Private + signed URL (GDPR).

</details>

</details>

---

← Назад: [01 — FR](01-functional-requirements.md) · [FRAMEWORK](../FRAMEWORK.md) · Дальше: [03 — API](03-api-design.md) →

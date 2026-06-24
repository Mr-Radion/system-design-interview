---
layer: pattern
steps: [4]
related:
  - constraints/latency-vs-throughput
  - data/pagination-cursor-offset
  - data/sharding-partitioning
  - data/normalization-denormalization
  - data/sql-vs-nosql-paradigm
  - technologies/databases
---

# Indexing Strategy (стратегия индексирования)

> **Главное:** индекс — не «да/нет», а **3 слоя**: gate → **алгоритм** (механика структуры) → **форма** (single / composite / partial / covering).  
> Вход — **FR** (шаг 1) + **NFR** (шаг 2 NFR). Выход — DDL на HLD §3.2.  
> Структура по мотивам Kleppmann, *Designing Data-Intensive Applications*, гл. 3.

## Цепочка решений (3 слоя)

```
Шаг 1 FR   →  equality / range / sort / keyword / semantic / geo
шаг 2 NFR  →  read:write · p99 · table size · write TPS
        ↓
HLD §3.2 / Deep Dive §4.2A     →  индекс нужен? (gate)
        ↓
HLD §3.2 / Deep Dive §4.2B     →  класс структуры (B-Tree / LSM / Hash / GIN / Vector / GiST / BRIN)
        ↓
HLD §3.2 / Deep Dive §4.2C     →  форма индекса
        ↓
Deep Dive §4.x → PostgreSQL · [partition](sharding-partitioning.md) · [databases](../technologies/databases.md)
```

## Сводная таблица (DDIA-style)

| Структура | Как ищет | Read | Write | Место | Типичный движок |
|-----------|----------|------|-------|-------|-----------------|
| **B-Tree** | range + sort + `=` | O(log N) | in-place, page split | среднее | PostgreSQL, MySQL |
| **LSM / SSTable** | sorted segments + bloom | O(log N) approx | append + merge | компакция | Cassandra, RocksDB, ES |
| **Hash** | bucket | O(1) avg | rehash | компактно | Redis, `USING HASH` |
| **Inverted (GIN)** | term → doc ids | fast keyword | heavy update | большой | PG GIN, Lucene |
| **Vector (ANN)** | embedding similarity | approximate k-NN | heavy build | очень большой | pgvector HNSW/IVFFlat, Milvus |
| **GiST** | R-tree-like | nearest | moderate | среднее | PostGIS |
| **BRIN** | block min/max | range on correlated cols | minimal | крошечный | PG BRIN, logs |

---

## Шаг A — gate: индекс нужен?

| Вопрос | Если «да» | Выбор |
|--------|-----------|-------|
| Запросы по колонке в WHERE / JOIN / ORDER BY? | Да | → шаг B |
| Таблица < 10K строк, rare reads? | Нет | seq scan OK |
| Write >> read, hot data в cache? | Нет | [cache](../architecture/caching-patterns.md) / [denorm](normalization-denormalization.md) |
| Append-only log, query по времени? | Частично | [partition](sharding-partitioning.md) + BRIN |

---

## Шаг B — дерево вопросов → алгоритм

```
1. Semantic / similarity по embedding?   да → Vector (HNSW / IVFFlat)
                                         нет → 2
2. Нужен range / ORDER BY?               нет → Hash (только =)
                                         да  → 3
3. Keyword / jsonb / full-text?            да → GIN tsvector
                                         нет → 4
4. Geo / ranges?                           да → GiST
                                         нет → 5
5. Append-only huge log?                   да → BRIN (+ partition)
                                         нет → 6
6. Write TPS > 10K, append-only?          да → LSM-СУБД ([sql-vs-nosql](sql-vs-nosql-paradigm.md))
                                         нет → B-Tree (default)
```

### GIN vs Vector — не путать на собесе

| Вопрос | GIN / full-text | Vector ANN |
|--------|-----------------|------------|
| Точное слово / токен? | да | нет |
| «Похожий смысл» / embedding? | нет | да |
| Explainable keyword match? | да | нет |

### Quick reference (FR + NFR)

| Запрос (FR) | NFR | Алгоритм |
|-------------|-----|----------|
| `=`, `IN`, JOIN, range, sort | default OLTP | B-Tree |
| только `=` | max point lookup | Hash |
| keyword, `jsonb @>` | search latency | GIN |
| semantic / RAG | recall@k, millions vectors | Vector HNSW |
| geo | location UC | GiST |
| time-series log | write-heavy | BRIN |
| hot subset | write cost | Partial B-Tree |

---

## Карточки алгоритмов

### B-Tree

- **Как работает:** сбалансированное дерево; данные в фиксированных **страницах**; insert/update **на месте**; при переполнении — split страницы + WAL.
- **Операции:** read O(log N) · write O(log N) + splits · space средний.
- **Когда:** OLTP, timeline `ORDER BY`, FK joins, 90% relational UC.
- **Когда нет:** append-only write flood → LSM лучше.
- **PostgreSQL:** default; `CREATE INDEX idx ON t(a, b DESC)`.

### LSM / SSTable

- **Как работает:** write в **memtable** → flush sorted **SSTable**-сегментов на диск → **compaction** (merge); read идёт через bloom filter + несколько сегментов.
- **Операции:** write O(1) append · read дороже (merge) · space — компакция.
- **Когда:** write >> read, append-only, миллионы w/s (metrics, events).
- **Когда нет:** нужны in-place UPDATE, сложные JOIN, strong OLTP — бери B-Tree (PG).
- **Аналог:** Cassandra, RocksDB, LevelDB, Elasticsearch segments → [sql-vs-nosql](sql-vs-nosql-paradigm.md).

### Hash

- **Как работает:** hash(key) → bucket → список коллизий; только **exact match**.
- **Операции:** read O(1) avg · write O(1) · нет range/sort.
- **Когда:** session lookup, immutable key, pure `=`.
- **Когда нет:** `BETWEEN`, `ORDER BY`, prefix search.
- **PostgreSQL:** `CREATE INDEX ... USING HASH(col)` — редко; B-Tree часто достаточно.

### Inverted — GIN (full-text / jsonb)

- **Как работает:** **posting list**: термин/token → список doc id; для jsonb — inverted по ключам/элементам.
- **Операции:** search fast · insert/update тяжёлые (перестройка lists).
- **Когда:** `@@` full-text, `jsonb @>`, arrays, trigram `LIKE`.
- **Когда нет:** semantic similarity (→ Vector); частые UPDATE поля.
- **PostgreSQL:** `USING GIN`; `to_tsvector` для full-text.

### Vector ANN — HNSW / IVFFlat

- **Как работает:** embedding (float[]) → **approximate** nearest neighbors; **HNSW** — многоуровневый граф соседей; **IVFFlat** — k-means кластеры → exact search внутри кластера.
- **Операции:** query O(log N) approximate · insert/update дороже HNSW · rebuild при смене модели.
- **Когда:** FR = похожие посты, semantic search, RAG; NFR = recall@k, p99 query, 100K+ vectors.
- **Когда нет:** keyword match (→ GIN); <100K vectors — brute force OK; нужен 100% exact recall.
- **PostgreSQL:** `pgvector` — `USING hnsw (embedding vector_cosine_ops)` или `ivfflat`; scale → Milvus, Pinecone.
- **HNSW vs IVFFlat:** HNSW — выше recall и query speed, больше RAM; IVFFlat — быстрее build, нужен tuning `lists`.

### GiST / SP-GiST

- **Как работает:** обобщённое **search tree**; R-tree для bounding boxes; SP-GiST — space partition (quadtrees).
- **Операции:** nearest-neighbor moderate · equality хуже B-Tree.
- **Когда:** `ST_DWithin`, ranges, PostGIS.
- **Когда нет:** simple `=` на scalar — B-Tree.
- **PostgreSQL:** `USING GIST(geom)`; PostGIS extension.

### BRIN

- **Как работает:** **block ranges** — min/max на группу страниц; skip целые блоки при range query.
- **Операции:** read O(blocks) · write minimal · tiny index size.
- **Когда:** append-only, `created_at` correlated с physical order, huge logs.
- **Когда нет:** random INSERT, low correlation — useless.
- **PostgreSQL:** `USING BRIN(created_at)`.

---

## Шаг C — форма индекса

| Форма | Когда | Пример |
|-------|-------|--------|
| **Single** | один dominant filter | `(post_id)` |
| **Composite** (concatenated, DDIA) | multi-column WHERE + sort | `(user_id, created_at DESC)` |
| **Covering (INCLUDE)** | fixed SELECT, index-only scan | `(user_id, created_at) INCLUDE (caption)` |
| **UNIQUE** | constraint + lookup | `(email)`, `(idempotency_key)` |
| **Partial** | hot subset | `WHERE status = 'active'` |

**Порядок в composite:** equality columns first → range/sort last.

**Multi-index vs composite:** два single `(a)` + `(b)` ≠ один `(a,b)` для `WHERE a AND b` — composite выигрывает.

**Leftmost prefix:** `(a,b,c)` для `WHERE a`, `WHERE a,b`; не для `WHERE b` alone.

---

## Связь с NFR и выбором СУБД

| NFR / нагрузка | Индекс / движок |
|----------------|-----------------|
| Read-heavy, p99 < 50ms | composite / covering под UC |
| Write-heavy (>1K w/s) | меньше индексов · partial · BRIN · LSM-СУБД |
| Large table (>100M rows) | composite + [partition](sharding-partitioning.md) |
| Low selectivity | partial, не full index на `status` |
| OLTP mixed | PG B-Tree |
| Semantic / RAG | pgvector HNSW или vector DB |

---

## Антипаттерны

| Ошибка | Почему плохо |
|--------|--------------|
| GIN вместо Vector для «похожий смысл» | keyword ≠ semantic |
| Vector для exact keyword | overkill, хуже explainability |
| Hash для feed timeline | нет ORDER BY |
| B-Tree на write flood 10K+ w/s | page splits; нужен LSM |
| GIN на часто UPDATE поле | write amplification |
| Composite `(created_at, user_id)` для `WHERE user_id` | leftmost prefix fail |

**Правило:** index for **read patterns from step 1 FR**, not «index everything».

---

## SQL-примеры

```sql
-- B-Tree composite (feed)
CREATE INDEX idx_posts_feed ON posts(user_id, created_at DESC);

-- Partial (saga active)
CREATE INDEX idx_saga_active ON saga_instances(instance_id)
  WHERE status IN ('PENDING', 'RUNNING');

-- GIN full-text
CREATE INDEX idx_posts_fts ON posts USING GIN(to_tsvector('russian', caption));

-- Vector semantic (pgvector)
CREATE INDEX idx_posts_emb ON posts
  USING hnsw (embedding vector_cosine_ops);
```

### Индекс + partition

Huge table → partition → меньший индекс на partition → [partition pruning](sharding-partitioning.md).

**Источники:** Kleppmann DDIA гл. 3 · [Habr ч.2](https://habr.com/ru/articles/877312/)

---

## Сокращения

| Аббревиатура | Расшифровка |
|--------------|-------------|
| FR | Functional Requirements |
| NFR | Non-Functional Requirements |
| ANN | Approximate Nearest Neighbor |
| LSM | Log-Structured Merge-tree |
| RAG | Retrieval-Augmented Generation |

Полный индекс: [FRAMEWORK.md](../../FRAMEWORK.md#trade-offs).

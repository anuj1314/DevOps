# Materialized Views in ClickHouse

## 1. Start Here — Regular Views

Before understanding Materialized Views, it helps to understand what a regular view is.

A regular view is nothing but a **saved SELECT query**. It stores no data:

```sql
CREATE VIEW active_users AS
SELECT user_id, count() AS total
FROM events
WHERE action = 'login'
GROUP BY user_id;
```

Every time you query `active_users`, ClickHouse runs that SELECT against `events` fresh. It is just a shortcut — no data is stored anywhere. On a table with billions of rows, this is slow every single time.

---

## 2. What a Materialized View Is

A Materialized View (MV) solves the performance problem by **pre-computing and storing results**. But ClickHouse MVs work differently from most databases — they do not recompute on read. Instead, they act as an **automatic trigger on insert**.

When new rows land in the source table, the MV:
1. Fires automatically
2. Runs its SELECT on **only the new rows**
3. Inserts the results into storage

```
INSERT INTO events (new rows arrive)
        ↓
MV trigger fires
        ↓
SELECT runs on only the new rows
        ↓
Results stored
```

The aggregation cost is paid **once at insert time**, not repeatedly at query time.

---

## 3. Why This Matters — Pre-aggregation

Without MV — expensive at query time:
```sql
-- Scans billions of rows every single time this query runs
SELECT user_id, count() AS total
FROM events
GROUP BY user_id;
```

With MV — expensive at insert time, cheap at query time:
```sql
-- MV already computed this incrementally as rows arrived
-- This hits a small pre-aggregated table
SELECT user_id, total FROM active_users_mv;
```

The larger your source table, the more valuable this becomes.

---

## 4. Two Types of Materialized Views

### Type 1 — MV manages its own storage

```sql
CREATE MATERIALIZED VIEW active_users_mv
ENGINE = SummingMergeTree()
ORDER BY user_id
AS
SELECT user_id, count() AS total
FROM events
GROUP BY user_id;
```

ClickHouse creates a **hidden backing table** behind the scenes to store the results. The MV name (`active_users_mv`) is a pointer to that hidden table.

You can query it directly like a normal table:

```sql
SELECT * FROM active_users_mv;
```

ClickHouse names the hidden backing table with a `.inner.` prefix internally — but you never need to touch it directly.

### Type 2 — MV writes to an existing table (`TO` syntax)

```sql
CREATE MATERIALIZED VIEW events_mv TO events AS
SELECT * FROM events_kafka;
```

Here the MV has **no storage of its own**. The `TO events` means all results are routed into a table you already created and control. The MV is purely a trigger — a pipe between the source and destination.

You query the destination table, not the MV:

```sql
SELECT * FROM events;    -- correct, this has the data
SELECT * FROM events_mv; -- not useful, no data here
```

---

## 5. Comparison of Both Types

| | Type 1 (own storage) | Type 2 (TO existing table) |
|---|---|---|
| Storage | Hidden backing table auto-created | Your own pre-created table |
| Query | Query the MV directly | Query the destination table |
| Control over storage | Limited | Full — you define the engine, keys, TTL |
| Typical use case | Pre-aggregations, summaries | Kafka pipeline, fan-out, transformations |

---

## 6. The Kafka Pipeline — Why Type 2 is Used There

In the Kafka pipeline, the MV connects the Kafka Engine table to the MergeTree storage table:

```
events_kafka (Kafka Engine)
        ↓
events_mv (Materialized View — Type 2, TO events)
        ↓
events (MergeTree — actual storage)
```

- `events_kafka` — not queryable, no storage, just a Kafka consumer
- `events_mv` — not queryable, just a trigger that routes rows
- `events` — **this is what you query**, permanent storage

The MV also gives you a place to transform data at ingest — filter bad rows, rename columns, cast types — before the data lands in storage:

```sql
CREATE MATERIALIZED VIEW events_mv TO events AS
SELECT
    event_id,
    event_time,
    user_id,
    lower(action) AS action   -- transform at ingest
FROM events_kafka
WHERE user_id > 0;            -- filter at ingest
```

---

## 7. One MV Per Source, or Many

A single source table can have **multiple MVs** attached to it. Each MV fires independently on every insert:

```sql
-- MV 1: route all events to main storage
CREATE MATERIALIZED VIEW events_mv TO events AS
SELECT * FROM events_kafka;

-- MV 2: route only errors to an error table
CREATE MATERIALIZED VIEW events_errors_mv TO events_errors AS
SELECT * FROM events_kafka
WHERE _error != '';
```

This is called **fan-out** — one source, multiple destinations with different transformations.

---

## 8. Important Gotcha — MVs Do Not Backfill

MVs only trigger on rows inserted **after the MV was created**. Existing data in the source table is not processed.

```
Table has 1 billion existing rows
        ↓
You create a Materialized View today
        ↓
Those 1 billion rows are NOT processed — MV storage is empty
        ↓
Only rows inserted from this point forward are picked up
```

To backfill existing data, you must do it manually:

```sql
INSERT INTO active_users_mv
SELECT user_id, count() AS total
FROM events
GROUP BY user_id;
```

---

## 9. Inspecting Materialized Views

```sql
-- See all materialized views in the system
SELECT name, engine, as_select
FROM system.tables
WHERE engine = 'MaterializedView';

-- See the hidden backing table for a Type 1 MV
SELECT name
FROM system.tables
WHERE name LIKE '.inner%';
```

---

## Summary

| Concept | Key Point |
|---|---|
| Regular view | Saved query — no storage, recomputes on every read |
| Materialized view | Trigger on insert — computes once, stores results |
| How it triggers | Fires automatically when new rows land in source table |
| What it processes | Only the newly inserted rows, not the entire table |
| Type 1 (own storage) | Query the MV directly |
| Type 2 (TO table) | Query the destination table, MV is just a trigger |
| Backfilling | Does not happen automatically — must be done manually |
| Fan-out | One source table can have multiple MVs for different destinations |

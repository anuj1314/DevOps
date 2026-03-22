# ReplacingMergeTree & Deduplication in ClickHouse

## What is ReplacingMergeTree?

`ReplacingMergeTree` is a specialized ClickHouse table engine that automatically removes duplicate rows during background merge operations. It is designed to simulate **upsert** (insert or update) behavior in a system that is otherwise append-only.

Duplicates are identified by the `ORDER BY` key — rows with identical values in all `ORDER BY` columns are considered duplicates.

## How It Works Internally

ClickHouse stores data in immutable **parts** on disk. Every INSERT creates a new part. Background **merge operations** periodically consolidate multiple parts into larger ones. During a merge, `ReplacingMergeTree` inspects rows with identical `ORDER BY` keys across the merging parts and keeps only one, discarding the rest.

Key points:
- Deduplication is **not immediate** — it happens when a merge occurs
- Merges are **partition-scoped**: rows in different partitions are never merged together
- Until a merge occurs, queries may return duplicate rows

## Basic Syntax

```sql
ENGINE = ReplacingMergeTree([version_column])
ORDER BY (key_columns)
```

### Without a version column

The last inserted row (by physical insertion order) wins.

```sql
CREATE TABLE page_views
(
    page_id   UInt64,
    title     String,
    views     UInt64
)
ENGINE = ReplacingMergeTree()
ORDER BY page_id;
```

### With a version column

The row with the **highest** version value wins. The version column must be a numeric type: `UInt*`, `Date`, `DateTime`, or `DateTime64`.

```sql
CREATE TABLE user_profiles
(
    user_id    UInt64,
    name       String,
    email      String,
    updated_at DateTime
)
ENGINE = ReplacingMergeTree(updated_at)
ORDER BY user_id;
```

## Benefits

| Benefit | Description |
|---|---|
| Simulates updates | Re-insert a row with the same key to "update" it |
| Reduces storage | Old versions are discarded after merges |
| CDC-friendly | Works well with Change Data Capture pipelines from OLTP databases |
| High write throughput | Writes are never blocked waiting for deduplication |
| No explicit UPDATEs needed | Avoids costly mutations on large datasets |

## What is Deduplication?

Deduplication is the process of ensuring that only one copy of a logical row exists in the table when multiple inserts share the same key.

In ClickHouse, deduplication is **eventual** — the system guarantees that duplicates will be removed at some point, but not immediately. This trade-off allows ClickHouse to ingest data at very high throughput.

## How to Query Deduplicated Data

### Option 1: `FINAL` keyword (simple, correct, slower)

The `FINAL` modifier forces deduplication at **query time**, guaranteeing you see only the latest version of each row regardless of whether background merges have run.

```sql
SELECT *
FROM user_profiles FINAL
WHERE user_id = 42;
```

- Always returns correct, deduplicated results
- Can be significantly slower on large tables (ClickHouse must merge data in memory)
- Works across partition boundaries (unlike background merges)

### Option 2: `OPTIMIZE TABLE` (force a merge manually)

```sql
-- Force merge all parts in the table
OPTIMIZE TABLE user_profiles FINAL;

-- Force merge a specific partition
OPTIMIZE TABLE user_profiles PARTITION '202603' FINAL;
```

- Triggers immediate deduplication
- Slow and resource-intensive — use only for batch jobs or one-off cleanup
- After this runs, queries without `FINAL` will also return correct results (until new inserts arrive)

### Option 3: Aggregation workaround (fastest, more complex)

For high-performance reads without `FINAL`, manually pick the latest row using aggregation:

```sql
SELECT
    user_id,
    argMax(name,       updated_at) AS name,
    argMax(email,      updated_at) AS email,
    max(updated_at)                AS updated_at
FROM user_profiles
GROUP BY user_id;
```

`argMax(column, version)` returns the value of `column` corresponding to the maximum `version` — effectively picking the latest row without a full merge.

## Practical Examples

### Inserting and updating a user profile

```sql
-- Initial insert
INSERT INTO user_profiles VALUES (1, 'Alice', 'alice@example.com', '2026-01-01 10:00:00');

-- "Update" by re-inserting with a newer timestamp
INSERT INTO user_profiles VALUES (1, 'Alice Smith', 'alice.smith@example.com', '2026-03-16 09:00:00');

-- Without FINAL — may return both rows until a merge happens
SELECT * FROM user_profiles WHERE user_id = 1;

-- With FINAL — always returns the latest version
SELECT * FROM user_profiles FINAL WHERE user_id = 1;
-- Result: (1, 'Alice Smith', 'alice.smith@example.com', '2026-03-16 09:00:00')
```

### Soft deletes

Use an `is_deleted` flag combined with the version column to handle deletions:

```sql
CREATE TABLE user_profiles
(
    user_id    UInt64,
    name       String,
    email      String,
    updated_at DateTime,
    is_deleted UInt8
)
ENGINE = ReplacingMergeTree(updated_at)
ORDER BY user_id;

-- Delete a user by inserting a tombstone row
INSERT INTO user_profiles VALUES (1, '', '', '2026-03-16 12:00:00', 1);

-- Query, excluding deleted rows
SELECT * FROM user_profiles FINAL WHERE is_deleted = 0;
```

## Limitations and Gotchas

| Issue | Explanation |
|---|---|
| No immediate deduplication | Duplicates persist until a background merge runs |
| Cross-partition duplicates | Rows in different partitions are never automatically merged |
| Historical data loss | Old versions are permanently deleted after a merge |
| `FINAL` performance cost | Can be up to 10x slower than a regular query on large datasets |
| `OPTIMIZE` is expensive | Forces merges — avoid running frequently in production |
| Non-idempotent defaults | Using `DEFAULT now()` generates unique rows on every insert, defeating deduplication |
| Merge size limits | Very large parts may stop being selected for merging, letting duplicates accumulate |

## When to Use ReplacingMergeTree

**Good fit:**
- Keeping the latest state of frequently updated entities (users, products, orders)
- CDC (Change Data Capture) ingestion from relational databases
- Device telemetry where only the current reading matters
- Any "current state" use case where history is not required

**Poor fit:**
- You need the full change history → use plain `MergeTree` and keep all rows
- You need strong immediate consistency → `ReplacingMergeTree` is eventually consistent
- Very high-frequency updates to the same key → consider `CollapsingMergeTree` instead

## Related Table Engines

| Engine | Use Case |
|---|---|
| `ReplacingMergeTree` | Keep latest version of a row by key |
| `CollapsingMergeTree` | Cancel out pairs of rows using a sign column |
| `VersionedCollapsingMergeTree` | Collapsing with out-of-order version support |
| `SummingMergeTree` | Automatically sum numeric columns during merges |
| `AggregatingMergeTree` | Store pre-computed aggregate states |

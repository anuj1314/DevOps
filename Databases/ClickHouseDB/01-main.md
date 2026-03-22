# ClickHouse Documentation

**Version: 26.2.4.23**

## What is ClickHouse?

ClickHouse is an open-source column-oriented database management system (DBMS) designed for online analytical processing (OLAP). It is built for high-performance analytical queries on large datasets.

## Key Concepts

### Column-Oriented Storage

Unlike traditional row-based databases, ClickHouse stores data column by column. This means:

- Only the columns needed for a query are read from disk
- Data within a column is of the same type, enabling better compression
- Aggregations and scans over large datasets are significantly faster

### MergeTree Engine

The primary table engine in ClickHouse. Data is written in parts and merged in the background.

```sql
CREATE TABLE events
(
    event_date Date,
    user_id    UInt64,
    action     String
)
ENGINE = MergeTree()
ORDER BY (event_date, user_id);
```

- `ORDER BY` defines the primary key and sort order
- Data is physically sorted on disk by the primary key
- Background merges keep data organized and efficient

### Partitioning

Tables can be partitioned to organize data into logical chunks (commonly by date):

```sql
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (event_date, user_id);
```

Partitioning allows ClickHouse to skip entire partitions when they don't match a query's filters.

## Basic Operations

### Insert Data

```sql
INSERT INTO events VALUES
    ('2026-03-16', 1001, 'login'),
    ('2026-03-16', 1002, 'purchase');
```

### Query Data

```sql
-- Count events per user
SELECT user_id, count() AS total
FROM events
WHERE event_date = '2026-03-16'
GROUP BY user_id
ORDER BY total DESC;
```

### Check Table Info

```sql
-- Show table structure
DESCRIBE TABLE events;

-- Show disk usage per partition
SELECT partition, rows, bytes_on_disk
FROM system.parts
WHERE table = 'events' AND active;
```

## How Queries Execute

1. **Query parsing** — SQL is parsed and validated
2. **Query planning** — ClickHouse builds an execution pipeline
3. **Parallel processing** — data is read and processed in parallel across CPU cores
4. **Merging results** — partial results from each thread are combined and returned

## Data Types

| Type | Description |
|---|---|
| `UInt8/16/32/64` | Unsigned integers |
| `Int8/16/32/64` | Signed integers |
| `Float32/64` | Floating point numbers |
| `String` | Variable-length string |
| `FixedString(n)` | Fixed-length string |
| `Date` | Date (no time) |
| `DateTime` | Date with time |
| `Array(T)` | Array of type T |
| `Nullable(T)` | Allows NULL values |

## Useful System Tables

```sql
-- List all databases
SELECT name FROM system.databases;

-- List all tables
SELECT database, name, engine FROM system.tables;

-- Monitor running queries
SELECT query_id, user, elapsed, query FROM system.processes;

-- Check ClickHouse version
SELECT version();
```

## Performance Tips

- Always filter on the primary key (ORDER BY columns) when possible — this enables index pruning
- Avoid `SELECT *` on wide tables; select only the columns you need
- Use `UInt` types instead of `Int` when values are non-negative — smaller storage footprint
- Batch inserts: prefer large bulk inserts over many small ones
- Use `FINAL` modifier with `ReplacingMergeTree` to deduplicate on read

## Common Table Engines

| Engine | Use Case |
|---|---|
| `MergeTree` | General-purpose analytics |
| `ReplacingMergeTree` | Deduplication by primary key |
| `SummingMergeTree` | Pre-aggregated sums |
| `AggregatingMergeTree` | Pre-computed aggregates |
| `ReplicatedMergeTree` | Replication across nodes |
| `Distributed` | Query across a cluster of shards |

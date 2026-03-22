# Kafka Engine & Materialized View Pipeline in ClickHouse

## Is This the Right Approach?

**Yes.** The pattern of **Kafka Engine → Materialized View → MergeTree table** is the officially recommended architecture for streaming Kafka data into ClickHouse. It is widely used in production and is considered the gold standard for native Kafka ingestion.

---

## How the Pipeline Works

```
Kafka Topic
    │
    ▼
┌─────────────────────────┐
│  Kafka Engine Table     │  ← Reads messages, never stores data
│  (events_kafka)         │
└─────────────┬───────────┘
              │  triggers automatically
              ▼
┌─────────────────────────┐
│  Materialized View      │  ← Transforms data, acts as a trigger
│  (events_mv)            │
└─────────────┬───────────┘
              │  inserts into
              ▼
┌─────────────────────────┐
│  MergeTree Table        │  ← Stores data permanently, queryable
│  (events)               │
└─────────────────────────┘
```

### Layer 1 — Kafka Engine Table

Acts as a gateway to your Kafka topic. It does **not store data**. ClickHouse starts background consumers that continuously poll the topic and present messages as rows.

- Cannot be queried directly with `SELECT`
- Consumer offsets are tracked by Kafka using the `kafka_group_name`
- Each INSERT to the Kafka engine table is handled by a background polling loop

### Layer 2 — Materialized View

Acts as the **trigger** between the Kafka engine and the storage table. When messages are polled from Kafka, the MV automatically runs its `SELECT` and writes results to the destination table.

- Supports filtering, renaming, type casting, and transformations
- One Kafka table can have multiple MVs (for different transformations/tables)
- The MV is not a separate storage — it just routes data

### Layer 3 — MergeTree Table

The actual storage. This is the table your application queries. Use the appropriate MergeTree variant:

- `MergeTree` — append-only events
- `ReplacingMergeTree` — deduplicate rows by key (for upserts)
- `ReplicatedMergeTree` — high availability with replication

---

## Complete Working Example

```sql
-- Step 1: Kafka Engine Table
CREATE TABLE events_kafka
(
    event_id   UUID,
    event_time DateTime,
    user_id    UInt64,
    action     String,
    properties String
)
ENGINE = Kafka
SETTINGS
    kafka_broker_list     = 'kafka-1:9092,kafka-2:9092',
    kafka_topic_list      = 'user-events',
    kafka_group_name      = 'clickhouse-consumer',
    kafka_format          = 'JSONEachRow',
    kafka_num_consumers   = 4,
    kafka_max_block_size  = 524288;

-- Step 2: Final storage table
CREATE TABLE events
(
    event_id   UUID,
    event_time DateTime,
    user_id    UInt64,
    action     LowCardinality(String),
    properties String
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_time)
ORDER BY (user_id, event_time, event_id)
TTL event_time + INTERVAL 90 DAY;

-- Step 3: Materialized View (connects the two)
CREATE MATERIALIZED VIEW events_mv TO events AS
SELECT
    event_id,
    event_time,
    user_id,
    action,
    properties
FROM events_kafka
WHERE user_id > 0;   -- optional filter/transform here
```

> **Order matters**: Create the Kafka table and the storage table first, then the MV. The moment the MV is created, consumption begins.

---

## Benefits of This Approach

| Benefit | Details |
|---|---|
| **Native & low latency** | Runs inside ClickHouse — no external process or hop |
| **Zero-copy routing** | Messages go directly from Kafka to storage without staging |
| **Transformation at ingest** | The MV SELECT can filter, cast, rename, and enrich data |
| **Fan-out** | One Kafka table → multiple MVs → multiple destination tables |
| **Automatic backpressure** | ClickHouse controls flush intervals; won't overwhelm the system |
| **Built-in offset tracking** | Kafka manages committed offsets per consumer group |
| **Low operational overhead** | No extra cluster, no external tools for basic use cases |
| **Parallel consumption** | Multiple consumers split partitions automatically |

---

## Error Handling & Dead Letter Queue

Enable `kafka_handle_error_mode = 'stream'` to capture malformed messages via virtual columns `_error` and `_raw_message`:

```sql
-- Kafka table with error streaming
CREATE TABLE events_kafka (...)
ENGINE = Kafka
SETTINGS
    kafka_format             = 'JSONEachRow',
    kafka_handle_error_mode  = 'stream',
    ...;

-- Dead letter table
CREATE TABLE events_errors
(
    error       String,
    raw_message String,
    received_at DateTime DEFAULT now()
)
ENGINE = MergeTree()
ORDER BY received_at;

-- MV to route bad messages to the error table
CREATE MATERIALIZED VIEW events_errors_mv TO events_errors AS
SELECT
    _error       AS error,
    _raw_message AS raw_message
FROM events_kafka
WHERE _error != '';
```

---

## Key Configuration Settings

| Setting | Default | Notes |
|---|---|---|
| `kafka_broker_list` | — | Comma-separated `host:port` list |
| `kafka_topic_list` | — | Comma-separated topic names |
| `kafka_group_name` | — | Consumer group for offset tracking |
| `kafka_format` | — | `JSONEachRow`, `CSV`, `Avro`, `Protobuf`, etc. |
| `kafka_num_consumers` | `1` | Set equal to number of topic partitions (max) |
| `kafka_max_block_size` | ~1M rows | Rows per flush to the MV; raise for throughput |
| `kafka_poll_timeout_ms` | `500` | How long to wait for a batch before flushing |
| `kafka_flush_interval_ms` | `7500` | Max time between flushes regardless of block size |
| `kafka_thread_per_consumer` | `false` | Set `true` when using multiple consumers |
| `kafka_handle_error_mode` | `default` | Use `stream` to capture parse errors |

**Rule of thumb**: `kafka_num_consumers` ≤ number of Kafka partitions ≤ CPU cores available.

---

## Delivery Semantics

The Kafka engine provides **at-least-once** delivery, not exactly-once. ClickHouse must:
1. Flush data to the MergeTree table (commit #1)
2. Commit the offset back to Kafka (commit #2)

If ClickHouse crashes between step 1 and 2, the same messages will be re-consumed on restart, causing duplicates.

**Mitigation — use `ReplacingMergeTree` as the final table:**

```sql
CREATE TABLE events
(
    event_id   UUID,
    event_time DateTime,
    user_id    UInt64,
    action     String
)
ENGINE = ReplacingMergeTree(event_time)
ORDER BY (event_id);  -- deduplicate by event_id
```

Query with `FINAL` to get clean results:

```sql
SELECT * FROM events FINAL
WHERE event_time > now() - INTERVAL 1 HOUR;
```

---

## Monitoring

```sql
-- Check consumer health and lag
SELECT
    database,
    table,
    consumer_id,
    assignments.topic_list,
    assignments.partition_id,
    assignments.current_offset,
    num_messages_read,
    last_exception
FROM system.kafka_consumers;

-- Check for messages in dead letter queue (if using dead_letter mode)
SELECT * FROM system.dead_letter_queue LIMIT 20;
```

---

## Distributed / Multi-Server Setup

For a ClickHouse cluster, create the Kafka engine table on **every node** using the same `kafka_group_name`. Kafka assigns different partitions to each node automatically. Each node writes to its local `ReplicatedMergeTree`.

```
Kafka Topic (8 partitions)
  ├── ClickHouse Node 1 → consumes partitions 0-1 → ReplicatedMergeTree
  ├── ClickHouse Node 2 → consumes partitions 2-3 → ReplicatedMergeTree
  ├── ClickHouse Node 3 → consumes partitions 4-5 → ReplicatedMergeTree
  └── ClickHouse Node 4 → consumes partitions 6-7 → ReplicatedMergeTree
```

---

## Important Gotchas

| Gotcha | What to do |
|---|---|
| Kafka table is not queryable with SELECT | Always access data through the final MergeTree table |
| First MV starts consuming immediately | Create **all** MVs before the Kafka table, or create them in one transaction |
| No DEFAULT values in Kafka table columns | All columns must come from the message payload |
| Multiple consumers need dedicated threads | Set `kafka_thread_per_consumer = 1` |
| Duplicates are possible on restart | Use `ReplacingMergeTree` + `FINAL` for idempotency |
| Consumers ≠ partitions → some are idle | Match `kafka_num_consumers` to partition count |

---

## Alternatives

| Tool | When to prefer it |
|---|---|
| **Kafka Connect ClickHouse Sink** | Need exactly-once guarantees; enterprise environments |
| **ClickPipes** (Cloud only) | Using ClickHouse Cloud and want a fully managed, zero-ops setup |
| **Apache Flink** | Complex transformations: windowing, stateful joins, multi-source aggregations |
| **Vector** | Lightweight pipelines; simple enrichment with low operational overhead |
| **Debezium + Kafka Connect** | CDC from a relational database into ClickHouse |
| **Timeplus Proton** | Purpose-built stream processor for ClickHouse, lighter than Flink |

**Summary**: For most self-hosted use cases with straightforward transformations, the native Kafka Engine pattern is the right choice. Reach for external tools only when you need exactly-once guarantees, complex stream processing, or a fully managed service.

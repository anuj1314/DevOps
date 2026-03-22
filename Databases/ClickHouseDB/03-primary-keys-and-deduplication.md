# Primary Keys and Deduplication in ClickHouse

## Primary Keys are Not Unique

In traditional databases (PostgreSQL, MySQL), a primary key must be unique and is used for row identification. In ClickHouse, the primary key works differently:

| | Traditional DB | ClickHouse |
|---|---|---|
| Primary key | Must be unique | Not enforced unique |
| Purpose | Row identification | Sorting + index pruning |
| Duplicates | Rejected | Allowed |

ClickHouse is an analytical database operating at massive scale — enforcing uniqueness on every insert would be too expensive. Instead, the primary key exists to **sort data on disk** and **build a sparse index for fast range scans**.

## How ORDER BY Defines the Primary Key

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

Both `event_date` and `user_id` form a **compound primary key**. Data is sorted lexicographically — first by `event_date`, then by `user_id` within each date:

```
event_date    user_id
----------    -------
2026-03-15    1001
2026-03-15    1003
2026-03-16    1001
2026-03-16    1002
2026-03-17    1002
```

## Index Pruning Works Left-to-Right

Because sorting is hierarchical, the index is only effective from left to right:

| Query filter | Index used? |
|---|---|
| `WHERE event_date = '2026-03-16'` | Yes — leftmost key, data is globally sorted by date |
| `WHERE event_date = '2026-03-16' AND user_id = 1001` | Yes — both keys narrow the range |
| `WHERE user_id = 1001` | No — `user_id` is not globally sorted, requires full scan |

Think of it like a phone book sorted by (LastName, FirstName):
- You can find all "Smiths" quickly — leftmost key works alone
- You can find "John Smith" quickly — both keys together work
- You can't find all "Johns" quickly — second key alone is useless without the first

## Deduplication with ReplacingMergeTree

Since ClickHouse allows duplicate primary keys, a specific engine exists to handle deduplication:

```sql
CREATE TABLE events
(
    event_date Date,
    user_id    UInt64,
    action     String,
    version    UInt64
)
ENGINE = ReplacingMergeTree(version)
ORDER BY (event_date, user_id);
```

### How Duplicates are Identified

Only the **ORDER BY columns** define what counts as a duplicate. All other columns are just payload — their values don't affect whether two rows are considered duplicates.

```
event_date    user_id    action      version
----------    -------    ------      -------
2026-03-16    1001       'login'     1
2026-03-16    1001       'purchase'  2   ← duplicate (same event_date + user_id)
```

Even though `action` is different, these two rows are duplicates because their primary key `(event_date, user_id)` matches.

### How the Winner is Decided (version column)

The `version` column passed to `ReplacingMergeTree(version)` decides which duplicate survives — the row with the **highest version value** is kept.

```
Before merge:
  (2026-03-16, 1001, 'login',    v=1)
  (2026-03-16, 1001, 'purchase', v=2)

After merge:
  (2026-03-16, 1001, 'purchase', v=2)   ← wins, higher version
```

Common choices for the version column:

| Column type | Example |
|---|---|
| Timestamp | `updated_at DateTime` — later time = higher value |
| Integer counter | `version UInt64` — manually incremented |

Without a version column (`ReplacingMergeTree()` with no argument), ClickHouse keeps the last row in physical order within the merged part — essentially arbitrary. **Always specify a version column.**

### How Deduplication Happens Efficiently (No Full Table Scan)

Deduplication does not scan the whole table. It happens **during background merges** of adjacent parts. Since parts are already sorted by the primary key, finding duplicates is just comparing adjacent rows — a linear walk through sorted data:

```
Part 1 (sorted):  1001/v1,  1003/v1,  1005/v1
Part 2 (sorted):  1001/v2,  1004/v1

Merge result:
  1001 → in both parts, keep higher version (v2)
  1003 → only in Part 1, keep it
  1004 → only in Part 2, keep it
  1005 → only in Part 1, keep it
```

The primary key identifies duplicates; the version column picks the winner.

### Deduplication is Eventual, Not Immediate

Merges happen in the background. Between inserts and the next merge, duplicate rows may coexist. To force deduplication at query time:

```sql
SELECT * FROM events FINAL
WHERE user_id = 1001;
```

`FINAL` deduplicates on read but is slower — use it only when strong consistency is required.

## Summary

| Concept | ClickHouse Behavior |
|---|---|
| Primary key uniqueness | Not enforced — duplicates allowed |
| Primary key purpose | Sort data + enable index pruning |
| Index pruning | Works left-to-right on ORDER BY columns |
| Duplicate identification | Only ORDER BY columns are compared |
| Duplicate resolution | Version column — highest value wins |
| When deduplication runs | During background merges (eventual) |
| Force immediate dedup | Use `SELECT ... FINAL` |

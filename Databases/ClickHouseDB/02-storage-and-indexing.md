# Storage and Indexing in ClickHouse

## 1. Column-Oriented Storage

Unlike row-based databases that store each row together, ClickHouse stores each column in its own file on disk:

```
part_dir/
  event_date.bin   ← all event_date values
  user_id.bin      ← all user_id values
  action.bin       ← all action values
```

When you query `SELECT user_id FROM events`, ClickHouse only opens `user_id.bin` — `event_date.bin` and `action.bin` are never touched. This is the core benefit of columnar storage: **read only what you need**.

However, column storage alone only solves *which columns* to read. It does nothing about *how many rows* to read within those columns.

---

## 2. The Row Filtering Problem

Even though each column is in its own file, the file still contains **all rows** sequentially:

```
user_id.bin:
[1001, 1003, 1001, 1002, 1005, 1001, 1004 ...]
 row0  row1  row2  row3  row4  row5  row6
```

For `WHERE user_id = 1001`, ClickHouse would have to read the entire file top to bottom to find matching rows. Column storage saved you from reading other column files, but not from scanning all rows in `user_id.bin`.

This is what the **primary key index** solves.

---

## 3. Primary Key Index — Reducing Rows to Read

When you define `ORDER BY (event_date, user_id)`, data inside each part is **physically sorted** by those columns. This means all rows for a given `event_date` are contiguous, and within that, rows are further sorted by `user_id`.

```
position:       0           1           2           3
event_date.bin: 2026-03-15  2026-03-15  2026-03-16  2026-03-16
user_id.bin:    1001        1002        1001        1002
```

ClickHouse builds a **sparse index** on top of this sorted data:

```
Index:
  row 0     → primary key value starts here
  row 8192  → primary key value starts here
  row 16384 → ...
```

For `WHERE event_date = '2026-03-16' AND user_id = 1001`, it consults the index, jumps directly to the relevant row range, and reads only that — no full scan needed.

Index pruning works **left-to-right** on the ORDER BY columns:

| Query filter | Index used? |
|---|---|
| `WHERE event_date = '2026-03-16'` | Yes — leftmost key, globally sorted |
| `WHERE event_date = '2026-03-16' AND user_id = 1001` | Yes — both keys narrow the range |
| `WHERE user_id = 1001` | No — not globally sorted, full scan required |

---

## 4. How Columns Stay in Sync — Row Position

Since each column lives in a separate file, you might wonder: how does ClickHouse know which `user_id` belongs to which `event_date`?

The answer is **row position**. Every `.bin` file stores values in the exact same order:

```
position:       0           1           2           3
                ↓           ↓           ↓           ↓
event_date.bin: 2026-03-15  2026-03-15  2026-03-16  2026-03-16
user_id.bin:    1001        1002        1001        1002
action.bin:     'login'     'purchase'  'login'     'purchase'
```

Row 2 in `event_date.bin` and row 2 in `user_id.bin` always belong to the same logical row. **Position is the implicit link** between column files.

---

## 5. Marks Files — Fast Seeking Within a Column

Knowing the row position isn't enough — ClickHouse still needs to jump to the right byte inside a `.bin` file without reading from the start. This is solved by **marks files**:

```
part_dir/
  event_date.bin   ← column data
  event_date.mrk   ← maps row positions → byte offsets in event_date.bin
  user_id.bin
  user_id.mrk
  action.bin
  action.mrk
```

The `.mrk` file acts as a lookup table: given a row position, it returns the exact byte offset to seek to in the `.bin` file. This makes jumping to any row position fast.

---

## 6. Partitioning — Physical Separation by Date

Partitioning adds a layer above all of this. Each partition gets its own directory on disk:

```
/data/events/
  202603/               ← partition for March 2026
    part_1/
      event_date.bin
      user_id.bin
      action.bin
  202602/               ← partition for February 2026
    part_1/
      event_date.bin
      user_id.bin
      action.bin
```

For a query filtering on March 2026, ClickHouse **never opens the `202602/` directory at all** — those files are completely skipped before any index or column logic runs.

---

## 7. All Three Levels Working Together

```sql
SELECT user_id FROM events
WHERE toYYYYMM(event_date) = 202603
  AND user_id = 1001;
```

| Level | What happens |
|---|---|
| Partition pruning | Skips all directories except `202603/` |
| Primary key index | Within `202603/`, jumps to row range matching `user_id=1001` |
| Marks file | Seeks directly to the right byte offset in `user_id.bin` |
| Column storage | Only reads `user_id.bin`, skips `event_date.bin` and `action.bin` |

Each level independently reduces work, and all three stack together.

---

## 8. Step-by-Step Query Execution

```sql
SELECT user_id FROM events WHERE event_date = '2026-03-16';
```

**Step 1** — Partition pruning: identify the correct partition directory.

**Step 2** — Primary key index: narrow down which parts and row ranges contain `event_date = '2026-03-16'`.

**Step 3** — Read `event_date.bin` for only that row range, find matching row positions:
```
position 0 → 2026-03-15  ✗
position 1 → 2026-03-15  ✗
position 2 → 2026-03-16  ✓
position 3 → 2026-03-16  ✓
matching positions: {2, 3}
```

**Step 4** — Use `user_id.mrk` to find the byte offsets for positions 2 and 3 in `user_id.bin`.

**Step 5** — Read only those bytes from `user_id.bin`:
```
position 2 → 1001
position 3 → 1002
```

**Step 6** — Return result. `action.bin` was never opened.

---

## Summary

| Concept | What it solves |
|---|---|
| Column storage | Read only the columns needed — skip irrelevant column files |
| Primary key index | Read only the rows needed — skip irrelevant row ranges |
| Row position | Keeps separate column files in sync — position is the implicit link |
| Marks files (.mrk) | Seek directly to any row position in a column file — no sequential scan |
| Partitioning | Skip entire date ranges — never open irrelevant partition directories |

---
title: SQLite and the Write-Ahead Log
description: Why a -wal file is evidence in its own right, what opening the database destroys, and how to read both states without losing deleted records.
tags:
  - forensics
  - guide
---

# SQLite and the Write-Ahead Log (WAL)

Every SQLite file on this page can sit next to a `-wal` and a `-shm`. Treat the `-wal` file as evidence in its own right,
not as a supplementary file; it may contain what you're looking for and could be destroyed during analysis.

!!! note "When this actually matters"

    **Most of the time it does not.** A browser that closed cleanly has an empty or absent WAL, and the database in front
    of you is the whole story.

    It matters when someone has been clearing up after themselves. The case to have in mind is an intrusion whose entry
    point is a URL like a phishing link, where the threat actor then clears browsing data to break the chain back to
    that first click. The database loses the row; the log can still hold it (`-wal`), right up until the moment
    you open the file.

    If you are in an environment with *a good maturity level* and have more than one disk or log to analyze (by which I
    mean you have proxy logs, DNS resolver logs, etc.) you can consider the WAL file as confirmation of the action.

**The WAL holds more than the database does.** it's not a buffer of pending writes. It stores successive versions
of database pages, so it carries records that were deleted before the last checkpoint, and it can carry an entire table
the main file has never seen.

**Opening the database destroys it.** When SQLite opens a database with a `-wal` beside it, it replays the log; when the
last connection closes, it checkpoints and then deletes both `-wal` and `-shm`. What survives is the final state. Every
intermediate page version, and anything deleted before that checkpoint, is gone from disk entirely. It never reaches the
database, because the database only ever receives the latest version of each page. Double-clicking an acquired `History`
in a SQLite browser is enough to do this, and nothing warns you.

For more on how it works, see SQLite's own [Write-Ahead Logging](https://sqlite.org/wal.html) documentation.

!!! danger "Measured on a reproduction"

    An acquired profile: `History` at 4 KB, `History-wal` at 20 KB. Read with the log ignored, the main file has no rows
    at all, not even the table: everything lived in the log. A URL had been inserted and then deleted before acquisition,
    and `strings` finds it in the `-wal`.

    Open that database once and close it. Result: `History` at 8 KB, no `-wal`, no `-shm`, one surviving row. The deleted
    URL is in neither file, and no longer anywhere on disk.

## Handling

To read the database while deliberately ignoring the `-wal`, use Python. SQLite's `immutable=1` URI parameter skips WAL
recovery, so it shows what the database alone contains:

```python
import sqlite3
db = sqlite3.connect("file:History?mode=ro&immutable=1", uri=True)
```

If you would rather not write code, DB Browser for SQLite gets you the same two states with two folders. Put a copy of
the database on its own in one folder, without the `-wal`, and open it. Then put the database and its `-wal` together in
a second folder and open that. The difference between the two reads is exactly what the log contributed. Work on copies
and the second read costs you nothing.

To read the `-wal` itself, run `strings` over it and search. If you are not sure what you are looking for, feed the
output to an LLM with a little context: it will do a first pass for you, and you will pick things up from it in return.

```bash
❯ strings -a History-wal | grep -Ei 'https?://'
```

What you will not get: timestamps, row boundaries, any way to tell a deleted record from a live one, or ordering. It
produces leads, not a timeline.

If you want to go further and recover deleted records (much as you would on a disk), two aging GitHub repositories do
the job. Both would benefit from a modern rewrite:

- [sqlparse](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser) (Python), by
  [Mari DeGrazia](https://www.sans.org/profiles/mari-degrazia)
- [undark](https://github.com/inflex/undark) (C)

!!! warning "VACUUM is what actually destroys deleted records"

    Deleted rows survive because SQLite only marks their pages as free and reuses them later.
    [`VACUUM`](https://www.sqlite.org/lang_vacuum.html) rebuilds the file from scratch, copying live rows into a fresh
    database and discarding the old one. The freelist collapses, and everything the two tools above would have carved
    goes with it.

    Never run it on evidence, and watch for GUI clients offering a compact or optimize button that runs it for you.
    Browsers vacuum their own databases as routine maintenance, so an empty freelist is not proof that anyone tampered
    with anything.

---

<small>
**Primary sources** ·
[SQLite WAL](https://sqlite.org/wal.html) and [`VACUUM`](https://www.sqlite.org/lang_vacuum.html)
</small>

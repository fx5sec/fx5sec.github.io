---
title: "FCSC 2025: iForensics - iNvisible"
description: Identify the recipient of an unsent message.
date: 2026-09-21
tags:
  - forensics
  - fcsc
---

# iForensics - iNvisible

## Challenge

A message seems to have failed to send. Find its recipient.

The flag format is `FCSC{<recipient>}`. For example: `FCSC{example@example.com}`.

**Difficulty:** ⭐⭐

[Official challenge on Hackropole](https://hackropole.fr/en/challenges/forensics/fcsc2025-forensics-iforensics-5/).

## Solution

I returned to the `sms.db` SQLite database from [iTreasure](itreasure.md). The iLEAPP report did not expose the recipient, so I inspected the underlying tables.

The [APOLLO `sms_chat.txt` query](https://github.com/mac4n6/APOLLO/blob/master/modules/sms_chat.txt) can also be used to list the stored messages.

### Check message status

I first looked for messages marked as undelivered:

```sql
SELECT * FROM message WHERE is_delivered = 0;
```

No results. Checking `is_finished = 0` and `is_prepared = 0` also returned nothing.

### Inspect the participants

The `handle` table provides another lead:

```sql
SELECT * FROM handle;
```

| Row ID | Identifier | Country | Service |
| --- | --- | --- | --- |
| 1 | kristy.friedman@outlook.com | us | SMS |
| 2 | robertswigert@icloud.com | us | iMessage |

To count stored messages linked to each handle:

```sql
SELECT h.id, COUNT(m.ROWID)
FROM handle h
LEFT JOIN message m ON m.handle_id = h.ROWID
GROUP BY h.ROWID;
```

The handle for `kristy.friedman@outlook.com` has no matching message rows. In the context of the challenge, this points to the recipient of the unsent message. The count establishes the absence of linked messages, but does not by itself prove that a send attempt failed.

**Flag:** `FCSC{kristy.friedman@outlook.com}`

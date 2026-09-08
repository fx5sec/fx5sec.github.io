---
title: Articles
description: Learn forensic methods through theory and practical labs.
icon: lucide/book-open
---

# Articles

Explanations of forensic artifacts, investigative methods, and the tools used to
examine them. Start with a question, learn how the evidence is produced, then
work through a lab to test what it can tell you.

## Choose the depth

| Depth | What to expect | Use it when |
| --- | --- | --- |
| **In-depth** | Theory, limitations, detailed workflows, and practical labs. | Learn and practice a method. |
| **Short article** | A focused explanation of one concept. | Fill a specific knowledge gap. |
| **Reference** | A compact lookup table. | Find an event ID or field. |

Depth describes the content, not the difficulty. The lab steps are exercises to
execute; record your Windows build, tool versions, and observations as you go.

## File systems and storage

| Article | Depth | What you will learn |
| --- | --- | --- |
| [USN Journal](../cheat-sheet/usn-journal.md) | In-depth | Collect and parse journal records, preserve identities, and assess missing history. |
| [SQLite and the WAL](../guides/sqlite-and-the-wal.md) | Short article | Understand why the database and its write-ahead log belong together. |

## Windows investigations

| Article | Depth | What you will learn |
| --- | --- | --- |
| [Windows Event IDs](../cheat-sheet/windows-event-ids.md) | Reference | Look up useful events by channel and investigation purpose. |

## Browser artifacts

The [Browser Artifacts collection](../guides/browser-artifacts/index.md) is an
in-depth series. Follow the chapters in order or choose the artifact you need:

| Article | Depth | What you will learn |
| --- | --- | --- |
| [Profiles and files](../guides/browser-artifacts/profiles.md) | In-depth | Locate the profile and preserve the files that belong together. |
| [WebCacheV01.dat](../guides/browser-artifacts/windows-webcache.md) | In-depth | Examine the Windows web cache and its surrounding context. |
| [Session files](../guides/browser-artifacts/sessions.md) | In-depth | Reconstruct tabs and sessions from retained state. |
| [Cache](../guides/browser-artifacts/cache.md) | In-depth | Work through cache collection, parsing, and interpretation. |
| [Timestamps](../guides/browser-artifacts/timestamps.md) | In-depth | Recognize and convert the time formats used by browser artifacts. |

## Memory and supporting artifacts

| Article | Depth | What you will learn |
| --- | --- | --- |
| [Volatility 3](../cheat-sheet/volatility3.md) | In-depth | Navigate process, network, and memory-analysis plugins. |


Quick lookup tables remain inside the relevant articles. The separate
[write-ups](../writeups/index.md) follow particular investigations and challenges.

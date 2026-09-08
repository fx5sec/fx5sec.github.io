---
title: Browser Timestamps
description: Convert browser timestamps without mixing epochs or units.
tags:
  - forensics
  - browser
  - guide
---

# Timestamps

Mixing epochs or units can shift an entire timeline by decades. Identify the field before converting its value.

!!! note "Safari lab verification"

    This information is currently based on documentation and older versions of macOS; lab verification (latest macOS + Safari versions) will be conducted shortly.

| Source | Epoch | Unit |
| --- | --- | --- |
| Chromium (`visit_time`, `last_visit_time`, cookies) | 1601-01-01 UTC | microseconds |
| Chromium `Bookmarks` JSON `date_added` | 1601-01-01 UTC | microseconds |
| Firefox (`visit_date`, `last_visit_date`, `dateAdded`) | 1970-01-01 UTC | microseconds |
| Firefox `logins.json` `timeCreated` | 1970-01-01 UTC | milliseconds |
| Safari (`visit_time`, binarycookies) | 2001-01-01 UTC | seconds, fractional |

The Chromium epoch is the Windows FILETIME epoch, and it's used on Linux and macOS
too. It is a property of the code, not of the platform. Safari history uses Apple's 2001 reference date,
with values in seconds rather than microseconds. The size of a number is a useful clue, not a reliable format test.

Convert directly in SQL, which avoids exporting and re-importing:

```sql
-- Chromium
SELECT datetime(visit_time/1000000 - 11644473600, 'unixepoch') AS visited, urls.url
FROM visits JOIN urls ON urls.id = visits.url
ORDER BY visit_time DESC;

-- Firefox
SELECT datetime(v.visit_date/1000000, 'unixepoch') AS visited, p.url
FROM moz_historyvisits v JOIN moz_places p ON p.id = v.place_id
ORDER BY v.visit_date DESC;

-- Safari
SELECT datetime(v.visit_time + 978307200, 'unixepoch') AS visited, i.url
FROM history_visits v JOIN history_items i ON i.id = v.history_item
ORDER BY v.visit_time DESC;
```

`11644473600` is the offset from 1601 to 1970, and `978307200` the offset from 1970
to 2001. Both are in seconds.

### Safari timestamps need a field name

The 2001 conversion above applies to `History.db` visit times. It is not a universal rule for every Safari artifact.
[APOLLO's Safari history module](https://github.com/mac4n6/APOLLO/blob/master/modules/safari_history.txt) uses this
offset for `history_visits.visit_time`; Apple defines the reference date in its
[Foundation documentation](https://developer.apple.com/documentation/foundation/date/timeintervalsincereferencedate-swift.property).

A plist parser may already return a date object rather than a number. Check that type before applying an offset.
Cache records and serialized session data need their own field definitions. In particular, a value from `Cache.db`
should not inherit the history conversion without checking its schema and representation.

Keep the raw value and state the output timezone. SQLite's `datetime()` examples return UTC to whole seconds, so
retain fractional source values when ordering events close together. Conversion also does not resolve attribution:
a synced Safari visit can have a valid timestamp without describing activity on the acquired Mac.

## Converting a single value

For the one timestamp you need to read right now, without writing SQL:

| Option | Where | Notes |
| --- | --- | --- |
| [DCode](https://www.digital-detective.net/dcode/) | Local, Windows | Handles all three epochs and many more. The reference tool for this |
| [EpochConverter](https://www.epochconverter.com/) | Web | Quick, but you are pasting case data into someone else's site. Think before you do |
| `date -d @<seconds>` | Local, Linux and macOS | Unix seconds only, so subtract the offset yourself first |
| Python | Local, anywhere | No install, no network, and it's three lines |

```python
from datetime import datetime, timedelta, timezone

chromium = lambda v: datetime(1601, 1, 1, tzinfo=timezone.utc) + timedelta(microseconds=v)
firefox  = lambda v: datetime(1970, 1, 1, tzinfo=timezone.utc) + timedelta(microseconds=v)
safari   = lambda v: datetime(2001, 1, 1, tzinfo=timezone.utc) + timedelta(seconds=v)
```

The local options are not just convenience. A timestamp pasted into a web converter is case data leaving your machine,
and on a client engagement that is a conversation you do not want to have.

---

<small>
**Primary sources** ·
[Chromium `base/time/time.h`](https://chromium.googlesource.com/chromium/src/+/main/base/time/time.h) for the 1601 epoch ·
[NSPR `prtime.h`](https://searchfox.org/mozilla-central/source/nsprpub/pr/include/prtime.h) for `PRTime`
</small>

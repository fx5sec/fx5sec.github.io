---
title: Browser Session Files
description: Recover tabs and navigation from browser session files.
tags:
  - forensics
  - browser
  - guide
---

# Session files

A history table tells you a page was visited. A session file tells you what was open at the same time, in which window,
and in what order the tabs got there.

They also outlive things they arguably should not. A tab open when the browser was killed can appear in a session file
with no matching history row, and **clearing browsing data does not necessarily rewrite them**.

This is also the machinery behind "continue where you left off" and crash recovery: the browser keeps these files
precisely so it can put the tabs back.

## Chromium

Modern builds keep them under `Sessions/` in the profile:

| File | Holds |
| --- | --- |
| `Session_<timestamp>` | Windows and tabs, with navigation entries per tab |
| `Tabs_<timestamp>` | Per-tab navigation, written more often than the session file |
| `Current Session`, `Current Tabs` | Older layout, directly in the profile root |
| `Last Session`, `Last Tabs` | The previous run, in that same older layout |

The numeric suffix distinguishes one session from another. Convert it before you rely on it, and sanity check the 
result against the file's own modification time rather than assuming which epoch it uses.

All of these are **SNSS**, a binary format that is not documented by Google:

```
[4 bytes]  magic "SNSS"
[4 bytes]  version, little-endian uint32
repeated:
  [2 bytes]  command size
  [1 byte]   command ID
  [n bytes]  payload, a serialized Chromium Pickle
```

```bash
❯ xxd Session_13432651959788794 | head -n 1
00000000: 534e 5353 0300 0000 0900 098c 387c 6a00  SNSS........8|j.
```

The commands that carry the evidence are `kCommandUpdateTabNavigation` (the URL and
title of a navigation), `kCommandSetTabWindow` (which window a tab belongs to),
`kCommandSetSelectedTabInIndex` (which tab was in front) and `kCommandTabClosed`.
That last one matters: the file records tabs closed *during* the session, so it holds
URLs the user deliberately shut.

Parsing them:

| Tool | Notes |
| --- | --- |
| [ccl_chromium_reader](https://github.com/cclgroupltd/ccl_chromium_reader) | Python 3.10+, partial SNSS support. Already a Hindsight dependency, so you likely have it |
| [Chromagnon](https://github.com/Squiblydoo/Chromagnon) | Parses session files alongside history, visited links and cache |
| [chrome-session-viewer](https://github.com/lachlanallison/chrome-session-viewer) | Browser-based viewer. There is a [live demo](https://lachlanallison.github.io/chrome-session-viewer), which is genuinely handy for a quick look |

`strings` works here too. It will give you URLs and titles out of the Pickle payloads and nothing about which window or 
tab they belonged to, which is precisely the part worth having.

## Firefox

| File | Holds |
| --- | --- |
| `sessionstore.jsonlz4` | Profile root. Written on a clean shutdown |
| `sessionstore-backups/recovery.jsonlz4` | The live session, rewritten while the browser runs |
| `sessionstore-backups/previous.jsonlz4` | The session before the current one |
| `sessionstore-backups/recovery.baklz4` | Backup of the recovery file |
| `sessionstore-backups/upgrade.jsonlz4-<build>` | Snapshot taken at a version upgrade |

Far friendlier than SNSS, once you get past the container. Decompress it and you have plain JSON: windows, tabs, and a
full back-forward entry list per tab including titles and referrers.

!!! warning
    The files listed above exist for a "standard" configuration; if Firefox is configured to clear any data upon closing, 
    only the `recovery.jsonlz4` and `recovery.baklz4` files will be found, and they will often be outdated.


### The mozLz4 container

Firefox wraps these files in its own container. It is **not** the LZ4 frame format, which is why the `lz4` command line
tool refuses them:

```
[8 bytes]  magic "mozLz40\0"
[4 bytes]  uncompressed size, little-endian uint32
[n bytes]  raw LZ4 block data
```

Demonstration: 
```bash
❯ xxd recovery.jsonlz4 | head -n 1
00000000: 6d6f 7a4c 7a34 3000 bc66 0000 f001 7b22  mozLz40..f....{"
```

- Magic: `6d6f 7a4c 7a34 3000`
- Uncompressed size: `66bc`, in decimal = 26300

Strip the twelve byte header and hand the rest to an LZ4 **block** decompressor:

```python title="read_mozlz4.py" linenums="1"
#!/usr/bin/env python3
"""Decompress a Firefox mozLz4 container and print the JSON it holds."""

import argparse
import json
import pathlib
import struct

import lz4.block  # pip install lz4

MAGIC = b"mozLz40\0"


def read_mozlz4(path):
    blob = pathlib.Path(path).read_bytes()
    if blob[:8] != MAGIC:
        raise ValueError(f"not a mozLz4 file, header is {blob[:8]!r}")
    size = struct.unpack("<I", blob[8:12])[0]
    return json.loads(lz4.block.decompress(blob[12:], uncompressed_size=size))


def main():
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("-f", "--file", required=True, help="path to the .jsonlz4 file")
    parser.add_argument("--urls", action="store_true", help="print titles and URLs only")
    args = parser.parse_args()

    data = read_mozlz4(args.file)
    if not args.urls:
        print(json.dumps(data, indent=2))
        return

    for window in data.get("windows", []):
        for tab in window.get("tabs", []):
            for entry in tab.get("entries", []):
                print(entry.get("title", ""), entry["url"])


if __name__ == "__main__":
    main()
```

```bash
❯ python3 read_mozlz4.py -f sessionstore-backups/recovery.jsonlz4          # full JSON
❯ python3 read_mozlz4.py -f sessionstore-backups/recovery.jsonlz4 --urls   # titles and URLs
```

The same function reads `bookmarkbackups/*.jsonlz4`, since the container is identical.

## Safari

!!! note "Safari lab verification"

    This information is currently based on documentation and older versions of macOS; lab verification (latest macOS + Safari versions) will be conducted shortly.

Start with the [Safari roots](profiles.md#find-every-profile), keeping the legacy and sandboxed locations separate.
`LastSession.plist` is useful on older acquisitions, but it is not a complete guide to modern Safari sessions.
Safari 15's move toward `SafariTabs.db` is documented in the [mac_apt project](https://github.com/ydkhatri/mac_apt/issues/78).

| Artifact | What to examine |
| --- | --- |
| `SafariTabs.db` | The `bookmarks` hierarchy: tab URLs, titles, parents and profile records |
| `BrowserState.db` | `tabs` and `tab_sessions`, including serialized per-tab history where present |
| `CloudTabs.db` | `cloud_tabs` and `cloud_tab_devices`; retain the associated device |
| `LastSession.plist` | Legacy `SessionWindows` and `TabStates` arrays |
| `RecentlyClosedTabs.plist` | Retained closed-tab or closed-window state |

These stores can coexist. Inventory them before choosing a parser, and preserve their SQLite sidecars.
A missing legacy plist is not evidence that no tabs were retained.

### Read the tab hierarchy

On a working copy of `SafariTabs.db`, inspect the schema and list URL-bearing records:

```sql
PRAGMA table_info(bookmarks);

SELECT id, parent, title, url
FROM bookmarks
WHERE url IS NOT NULL AND url <> '';
```

Keep `parent`: flattening the rows discards the hierarchy needed to associate tabs with groups or profiles.
The query is an inventory, not a chronological list of visits or proof that all listed tabs were open together.

The [mac_apt SAFARI parser](https://github.com/ydkhatri/mac_apt/blob/master/plugins/safari.py) also reads timestamps
from serialized `local_attributes`. For `BrowserState.db`, it joins tabs to session data and extracts
`SessionHistoryEntries` where available. Safari can therefore retain per-tab navigation history; do not stop at
current tab URLs. Use a parser matched to the schema rather than treating the session BLOB as plain text.

### Separate synced state from local activity

Apple's [iCloud Tabs documentation](https://support.apple.com/en-us/108409) describes access to tabs from other
signed-in devices. A cloud-tab URL on the acquired Mac may describe activity elsewhere. Preserve the device name
and identifier, and distinguish synchronization or state timestamps from a local navigation time.

For legacy plists, `plutil -p` or a plist parser exposes the stored arrays. Whichever format you read, correlate
session state with history and cache records before describing when a page was opened or viewed.

---

<small>
**Further reading** ·
[Reverse engineering the SNSS format](https://github.com/JRBANCEL/Chromagnon/wiki/Reverse-Engineering-SSNS-Format)
</small>

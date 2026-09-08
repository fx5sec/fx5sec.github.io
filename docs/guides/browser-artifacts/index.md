---
title: Browser Artifacts
description: Find and interpret browser evidence across platforms.
tags:
  - forensics
  - browser
  - guide
---

# Browser Artifacts

Browser artifacts can help reconstruct navigation, recover cached content, and identify tabs or local files
associated with an activity. Start with the profile, then choose the artifact that answers your question.

The guide covers Chromium-based browsers, Firefox and related browsers, and desktop Safari on macOS. Shared code
does not guarantee identical paths, settings, or retention behavior across products and versions.

!!! note "Safari lab verification"

    This information is currently based on documentation and older versions of macOS; lab verification (latest macOS + Safari versions) will be conducted shortly.

!!! info "Scope and validation"

    Lab examples identify the tested environment where available. Other sections, including the Safari additions,
    draw on the linked documentation and parser source. They have not all been reproduced in the lab.

    Paths and schemas change. Record the browser version, preserve the source files, and check the fields before
    applying a query. A stored URL alone does not establish who opened it or whether its content was viewed.

    An LLM helped with translation, formatting, and drafting. Sources are linked beside the relevant claims or at
    the end of each page.

For tools, jump to the [list below](#tools).

<div class="grid cards" markdown>

-   :lucide-folder-tree:{ .lg .middle } __Profiles and files__

    ---

    Profile roots for the Chromium family, Firefox and Safari on the three operating
    systems, the files worth opening in each, and what snap and flatpak do to those
    paths on Linux.

    [:octicons-arrow-right-24: Read it](profiles.md)

-   :lucide-database:{ .lg .middle } __WebCacheV01.dat__

    ---

    A Windows ESE database with evidence beyond legacy browsing. Lab tests of local
    file activity, the gaps in its records, and how to collect it while it is locked.

    [:octicons-arrow-right-24: Read it](windows-webcache.md)

-   :lucide-panels-top-left:{ .lg .middle } __Session files__

    ---

    What was open at the same time, in which window and in what order the tabs got
    there. Chromium SNSS, Firefox mozLz4, and Safari's tab databases and legacy plists.

    [:octicons-arrow-right-24: Read it](sessions.md)

-   :lucide-archive:{ .lg .middle } __Cache__

    ---

    Recover response bodies and interpret their metadata. Chromium backends,
    Firefox cache2, and Safari's WebKit NetworkCache and legacy `Cache.db`.

    [:octicons-arrow-right-24: Read it](cache.md)

-   :lucide-clock:{ .lg .middle } __Timestamps__

    ---

    Three epochs, 1601, 1970 and 2001, and the offsets between them. Getting this wrong
    shifts a whole timeline by decades.

    [:octicons-arrow-right-24: Read it](timestamps.md)
</div>

## Tools

A list of tools that may be useful if you don't want to manually browse through all the tables or write SQL queries
to cross-reference multiple tables.

| Tool | Covers | Platform | How |
| --- | --- | --- | --- |
| [Hindsight](https://github.com/RyanDFIR/hindsight) | Chromium family and Firefox | Windows, macOS, Linux | `pip install pyhindsight`. CLI, or `hindsight_gui.py` for a web UI. Windows also has a standalone `hindsight_gui.exe`. Outputs XLSX, SQLite or JSONL |
| [NirSoft browser tools](https://www.nirsoft.net/web_browser_tools.html) | One tool per browser and per artifact | **Windows only** | Standalone executables, no install |
| [libesedb](https://github.com/libyal/libesedb) (`esedbexport`) | `WebCacheV01.dat` and any ESE database | Linux, macOS, Windows | Build from source, or a distribution package |
| [DB Browser for SQLite](https://sqlitebrowser.org/) | Any SQLite artifact | Windows, macOS, Linux | Use it to read, not to write |
| [APOLLO](https://github.com/mac4n6/APOLLO) | Apple SQLite artifacts, 247 modules | Windows, macOS, Linux | Python. Queries versioned per OS release |
| [mac_apt SAFARI plugin](https://github.com/ydkhatri/mac_apt/blob/master/plugins/safari.py) | Safari history, profiles, tabs and plists | Python, with the project's dependencies | Preserve the source paths in its output; check parser support against the acquired schema |
| [libplist](https://github.com/libimobiledevice/libplist) (`plistutil`) | Safari `.plist` files | Linux, macOS, Windows | Distribution package. `plutil -p` is already there on macOS |

<small>
**Further reading** ·
[Hindsight parses Firefox](https://dfir.blog/hindsight-parses-firefox/) ·
[forensics.wiki on Google Chrome](https://forensics.wiki/google_chrome/)
</small>

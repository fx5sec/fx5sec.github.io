---
title: Browser Profiles and Files
description: Locate browser profiles and the evidence they contain.
tags:
  - forensics
  - guide
  - browser
---

# Profiles and files

Where each browser keeps its data, and which files in that directory are worth opening.

Work through it in order. Identify the profile first, since every path further down is
relative to one, then take the section for the browser family in front of you. If the
host is Linux and the browser was installed as a snap or a flatpak, the roots given per
family do not apply: the last section has the ones that do.

## Find the profile first

Never assume `Default`. A user with two profiles has `Default` and `Profile 1`, and
enterprise policy or a packaged build can move the root entirely. Settle this before
anything else, because everything below hangs off the profile you pick.

On a live system, the browser names its own path:

| Browser | Page |
| --- | --- |
| Chrome | `chrome://version` |
| Edge | `edge://version` |
| Brave | `brave://version` |
| Opera | `opera:about` |
| Firefox | `about:profiles` |

On an image, enumerate the profile directories under the roots below, then read `Local State` (Chromium) or `profiles.ini` (Firefox) to map directory names to
profile names and available account metadata. Safari 17 and later also support multiple profiles; see
[Safari](#safari) for its separate discovery process.

The Bash and macOS snippets inspect the current user's home. PowerShell enumerates `C:\Users`.

=== "Bash"

    ```bash  linenums="1"
    # Chromium family, live host
    ls -d ~/.config/google-chrome/*/ ~/.config/chromium/*/ \
          ~/.config/microsoft-edge/*/ ~/.config/BraveSoftware/Brave-Browser/*/ \
          ~/.config/opera/ 2>/dev/null

    # Firefox, live host
    ls -d ~/.mozilla/firefox/*/ 2>/dev/null

    # Snap and flatpak, which the two globs above never reach
    ls -d ~/snap/*/common/ ~/.snap/data/*/common/ ~/.var/app/*/ 2>/dev/null
    ```

=== "PowerShell"

    ```powershell  linenums="1"
    # Chromium family: Chrome, Edge, Brave, Opera
    Get-ChildItem C:\Users -Directory | ForEach-Object {
        $user = $_.Name
        @(
            "AppData\Local\Google\Chrome\User Data"
            "AppData\Local\Microsoft\Edge\User Data"
            "AppData\Local\BraveSoftware\Brave-Browser\User Data"
            "AppData\Roaming\Opera Software"
        ) | ForEach-Object {
            Get-ChildItem "C:\Users\$user\$_" -Directory -ErrorAction SilentlyContinue |
                Select-Object @{ n = 'User'; e = { $user } }, FullName
        }
    }

    # Firefox
    Get-ChildItem C:\Users -Directory | ForEach-Object {
        Get-ChildItem "C:\Users\$($_.Name)\AppData\Roaming\Mozilla\Firefox\Profiles" `
            -Directory -ErrorAction SilentlyContinue
    }
    ```

=== "macOS"

    ```bash  linenums="1"
    cd ~/Library/Application\ Support

    # Chromium family
    ls -d Google/Chrome/*/ Microsoft\ Edge/*/ BraveSoftware/Brave-Browser/*/ \
          com.operasoftware.Opera/ 2>/dev/null

    # Firefox
    ls -d Firefox/Profiles/*/ 2>/dev/null
    ```

Or you can search for the files directly: 

=== "Bash"

    ```bash
    find <path> -type f \( -name 'History' -o -name 'places.sqlite' -o -name 'History.db' \) 2>/dev/null
    ```

=== "PowerShell"

    ```powershell
    Get-ChildItem -Path "<path>" -File -Recurse -ErrorAction SilentlyContinue |
    Where-Object { $_.Name -in @("History", "places.sqlite", "History.db") }
    ```

---


## Chromium family

### Profile roots

The Chrome, Edge and Brave paths below are the **user data directory**. Each profile is a subdirectory of it, usually
`Default`, then `Profile 1`, `Profile 2` and so on.

| Browser | Windows | macOS | Linux |
| --- | --- | --- | --- |
| Chrome | `%LOCALAPPDATA%\Google\Chrome\User Data` | `~/Library/Application Support/Google/Chrome` | `~/.config/google-chrome` |
| Edge | `%LOCALAPPDATA%\Microsoft\Edge\User Data` | `~/Library/Application Support/Microsoft Edge` | `~/.config/microsoft-edge` |
| Brave | `%LOCALAPPDATA%\BraveSoftware\Brave-Browser\User Data` | `~/Library/Application Support/BraveSoftware/Brave-Browser` | `~/.config/BraveSoftware/Brave-Browser` |
| Opera | `%APPDATA%\Opera Software\Opera Stable` | `~/Library/Application Support/com.operasoftware.Opera` | `~/.config/opera(-gx)` |

Opera breaks the pattern twice on Windows. It sits under **roaming** `%APPDATA%`, not `%LOCALAPPDATA%`, and `Opera Stable` **is**
the profile directory: there is no `User Data` parent and no `Default` subdirectory. Its cache lives separately under
`%LOCALAPPDATA%\Opera Software\Opera Stable\Default\Cache`.

The Linux column assumes a distribution package. See
[Linux: snap and flatpak move the root](#linux-snap-and-flatpak-move-the-root).

### Profile contents

This layout is shared by Chrome, Edge, Brave and Opera. Where a browser diverges, it
is called out in the next section.

The SQLite files below all follow the `-wal` and `-shm` rules set out in
[SQLite and the Write-Ahead Log](../sqlite-and-the-wal.md).

These are the ones to open in almost every case.

| File | Format | Holds |
| --- | --- | --- |
| `History` | SQLite | Visits, downloads, search terms. The core of the timeline |
| `Network/Cookies` | SQLite | Cookies. Moved under `Network/` in recent versions |
| `Login Data` | SQLite | Saved credentials, passwords encrypted |
| `Web Data` | SQLite | Autofill entries, saved form values, payment cards |
| `Bookmarks` | JSON | Bookmarks, with creation timestamps |
| `Preferences` | JSON | Per-profile settings, extension list, download directory |
| `Local State` | JSON | **Sits in the user data directory**, not the profile. Profile to account mapping, and the DPAPI-wrapped key protecting cookies and passwords |
| `Sessions/` | SNSS | Open tabs and window state, current and previous |
| `Cache/Cache_Data/` | blockfile | Cached responses |
| `Extensions/` | Files | Installed extensions, one directory per ID |
| `Favicons` | SQLite | Icon per site. Survives history deletion in some flows |
| `Shortcuts` | SQLite | Omnibox text to destination mappings, so what the user typed |
| `Local Storage/leveldb/` | LevelDB | Per-origin web storage |
| `IndexedDB/` | LevelDB | Structured per-origin storage, often the richest of the three |

Everything else in the profile is situational. `Top Sites`,`Network Action Predictor` and `Media History` describe 
habits rather than events. `Secure Preferences`,`Network Persistent State` and `Network/Reporting and NEL` hold 
configuration and connection state. `Session Storage/`and `Sync Data/` are LevelDB stores worth opening only with a 
specific question already in hand.

Chromium documents none of these individually. [forensics.wiki](https://forensics.wiki/google_chrome/) keeps the most
complete community inventory if you end up needing one.

!!! warning 
    For Edge, on an acquisition predating June 2026 you may still find `Collections\collectionsSQLite` in the profile, holding saved
    pages and notes with their timestamps. Edge [retired Collections](https://support.microsoft.com/en-us/edge/organize-your-ideas-with-collections-in-microsoft-edge)
    in version 149, released June 4, 2026, and only saved page URLs were migrated into Favorites, so on an older image that
    database is the only place the notes and images existed.


### History tables worth knowing

| Table | Holds |
| --- | --- |
| `urls` | One row per distinct URL, with `visit_count` and `last_visit_time` |
| `visits` | One row per visit, with `visit_time`, `from_visit` and `transition` |
| `downloads` | One row per download, with target path, size and state |
| `downloads_url_chains` | The full redirect chain of a download |
| `keyword_search_terms` | Search terms typed into the omnibox |

### Following a visit chain

`visits.from_visit` refers to another row in `visits`, not to a row in `urls`. It links a visit to its recorded
predecessor. Use it to reconstruct a chain, then read `transition` to interpret the navigation.

Chromium stores a core type in the low eight bits of `transition`, with qualifiers in the remaining bits:

| Expression | Meaning |
| --- | --- |
| `transition & 255` | Core type: `0` = link, `1` = typed or another explicit navigation, `8` = reload |
| `transition & 1073741824` | Client redirect flag (`0x40000000`) |
| `transition & 2147483648` | Server redirect flag (`0x80000000`) |

The flags can coexist with a core type. Comparing the entire value with `1`, for example, misses typed transitions
that also carry qualifiers. Chromium's [transition definitions](https://chromium.googlesource.com/chromium/src/+/main/ui/base/page_transition_types.h)
describe the other types and flags.

This query shows each visit beside its recorded predecessor. Run it against a working copy of `History`:

```sql
SELECT v.id AS visit_id,
       u.url AS url,
       v.visit_time,
       v.from_visit AS predecessor_id,
       previous_url.url AS predecessor_url,
       v.transition AS transition_raw,
       v.transition & 255 AS transition_core,
       (v.transition & 1073741824) != 0 AS client_redirect,
       (v.transition & 2147483648) != 0 AS server_redirect
FROM visits AS v
JOIN urls AS u ON u.id = v.url
LEFT JOIN visits AS previous ON previous.id = v.from_visit
LEFT JOIN urls AS previous_url ON previous_url.id = previous.url
ORDER BY v.visit_time, v.id;
```

A missing predecessor does not prove that a URL was typed. The link may be unset or its target may no longer be
available. Likewise, a typed transition is evidence of a navigation category, not a keystroke log. Correlate the
chain with session data and other activity before attributing intent.

## Mozilla family

### Profile roots

Firefox splits its data across two roots. The roaming one holds the evidence, the local one holds the cache.

| Role | Windows | macOS | Linux |
| --- | --- | --- | --- |
| Profile | `%APPDATA%\Mozilla\Firefox\Profiles` | `~/Library/Application Support/Firefox/Profiles` | `~/.mozilla/firefox` |
| Cache | `%LOCALAPPDATA%\Mozilla\Firefox\Profiles` | `~/Library/Caches/Firefox/Profiles` | `~/.cache/mozilla/firefox` |

The Linux column assumes a distribution package. See
[Linux: snap and flatpak move the root](#linux-snap-and-flatpak-move-the-root).

Profile directories are named `<random>.<name>`, for example `k3f9a1zx.default-release`. `profiles.ini`, one level
above `Profiles`, maps them and marks the default.

```ini title="profiles.ini (example, fictional profile IDs)"
[Install4F96D1932A9F858E]
Default=Profiles/k3f9a1zx.default-release
Locked=1

[Profile0]
Name=default-release
IsRelative=1
Path=Profiles/k3f9a1zx.default-release

[Profile1]
Name=default
IsRelative=1
Path=Profiles/8h2m4p0q.default
Default=1

[General]
StartWithLastProfile=1
Version=2
```

The `[Install...]` section is authoritative on modern Firefox: its `Default=` names the profile the installation 
actually launches. The legacy `Default=1` inside a `[ProfileN]` block often points somewhere else entirely, and on the 
example above it does. The `ProfileN` numbering is allocation order, not recency, so `Profile0` is not necessarily 
the one in use.

Tor Browser and other Firefox forks reuse this layout inside their own application directory, so the file names below
still apply.

### Profile contents

These are the ones to open in almost every case.

| File | Format | Holds |
| --- | --- | --- |
| `places.sqlite` | SQLite | History and bookmarks together |
| `cookies.sqlite` | SQLite | Cookies |
| `favicons.sqlite` | SQLite | Site icons |
| `formhistory.sqlite` | SQLite | Saved form values |
| `logins.json` | JSON | Saved credentials, encrypted |
| `key4.db` | SQLite | The key material protecting `logins.json` |
| `prefs.js` | JS | Every non-default setting, including the download directory |
| `extensions.json` | JSON | Installed extensions with install timestamps |
| `sessionstore-backups/` | LZ4 JSON | Sessions. See [Session files](sessions.md#the-mozlz4-container) |
| `bookmarkbackups/` | LZ4 JSON | Dated bookmark snapshots, which outlive deletions |
| `cache2/entries/` | Files | Cached responses (`~/.cache/mozilla/firefox`) |
| `storage/default/` | Files | Per-origin storage, including IndexedDB |

The rest is situational: `permissions.sqlite` and `protections.sqlite` for per-site grants and blocked trackers,
`content-prefs.sqlite` for per-site settings such as zoom, `webappsstore.sqlite` and `storage-sync-v2.sqlite` for web
and extension storage, `cert9.db` for the certificate store, and `handlers.json` for protocol and file type handlers.

Mozilla documents the full profile layout in [What information is stored in my profile](https://support.mozilla.org/en-US/kb/profiles-where-firefox-stores-user-data#w_what-information-is-stored-in-my-profile).

### places.sqlite tables worth knowing

| Table | Holds |
| --- | --- |
| `moz_places` | One row per URL, with `visit_count`, `last_visit_date`, `title` |
| `moz_historyvisits` | One row per visit, with `visit_date`, `from_visit`, `visit_type` |
| `moz_bookmarks` | Bookmark tree, with `dateAdded` and `lastModified` |
| `moz_annos` | Annotations, including download metadata on older versions |
| `moz_origins` | Origins, with a frecency score per origin |

Anything ending in `.jsonlz4` or `lz4` in a Firefox profile is compressed with Mozilla's own container, not standard
LZ4. The format and how to unpack it are covered under [Session files](sessions.md#the-mozlz4-container).

## Safari

!!! note "Safari lab verification"

    This information is currently based on documentation and older versions of macOS; lab verification (latest macOS + Safari versions) will be conducted shortly.

This section covers desktop Safari on macOS. Since Safari 17, one macOS account can have several Safari profiles,
with separate history, cookies, website data, and Tab Groups. Bookmarks remain available across profiles, while
Favorites can use different folders. A bookmark alone therefore does not identify the profile used to visit it.
See [Apple's profile documentation](https://support.apple.com/en-ie/105100).

### Find every profile

Check both roots below in the acquired user's home. Older files may remain after an upgrade; their presence does
not establish that the current browser still writes them.

| Root | What to check |
| --- | --- |
| `~/Library/Safari/` | Traditional history, bookmarks, downloads and session artifacts |
| `~/Library/Containers/com.apple.Safari/Data/Library/Safari/` | Sandboxed Safari data, including modern tab databases and `Profiles/` |

Under the sandboxed root, inspect `Profiles/<UUID>/History.db` as well as the root `History.db`.
The [mac_apt SAFARI parser](https://github.com/ydkhatri/mac_apt/blob/master/plugins/safari.py) maps profile UUIDs to
names using `SafariTabs.db`: profile records in `bookmarks` have `parent = 0`, `type = 1`, and `subtype = 2`;
`external_uuid` identifies the profile and `title` gives its name. Check the schema before using those fields.

Retain the macOS user, source path, and profile UUID with each export. Combining several files named `History.db`
without those labels makes later attribution unnecessarily difficult. Preserve SQLite sidecars as described in
[SQLite and the Write-Ahead Log](../sqlite-and-the-wal.md), and run queries on working copies.

### Files worth collecting

Paths below are relative to the applicable Safari root unless stated otherwise. Availability depends on version.

| File | Format | Holds |
| --- | --- | --- |
| `History.db`, `Profiles/<UUID>/History.db` | SQLite | URLs and individual visits |
| `Bookmarks.plist` | plist | Bookmarks and Reading List entries |
| `Downloads.plist` | plist | Download URLs and destination paths |
| `SafariTabs.db` | SQLite | Tab and profile records |
| `BrowserState.db` | SQLite | Tab state and serialized session history, where present |
| `CloudTabs.db` | SQLite | Synced tabs and device records |
| `LastSession.plist`, `RecentlyClosedTabs.plist` | plist | Legacy session and closed-tab records |
| `TopSites.plist` | plist | Legacy frequently visited site records |

For a quick look at an acquired plist on macOS:

```bash
plutil -p "/path/to/working-copy/Downloads.plist"
```

Keep download records separate from proof that a file was opened. Check the destination file and other local
artifacts when that distinction matters. Session interpretation is covered under [Session files](sessions.md#safari).

Cookies and caches can sit outside these roots. Look for `Cookies.binarycookies` under `~/Library/Cookies/` and
within the acquired Safari container rather than assuming it accompanies `History.db`. A cookie records stored
site state, not a page view. For response bodies, follow [Cache](cache.md#safari).

### Reading History.db properly

For ready-made Safari queries, start with Sarah Edwards' [APOLLO history module](https://github.com/mac4n6/APOLLO/blob/master/modules/safari_history.txt).
Its supported schemas are versioned; use them as a reference, not a guarantee that every column exists in a newer
acquisition. Start by inspecting the two tables:

```sql
PRAGMA table_info(history_items);
PRAGMA table_info(history_visits);
```

`history_items` holds URLs; `history_visits.history_item` links each visit to its URL. Where the fields below exist,
keep them in the export:

| Column | Why it matters |
| --- | --- |
| `history_visits.origin` | APOLLO interprets `0` as local and `1` as iCloud-synced. Preserve unfamiliar values rather than assigning them to a device |
| `redirect_source`, `redirect_destination` | Visit identifiers used to reconstruct redirects |
| `load_successful` | Whether Safari recorded a successful load; it does not establish that the user read the page |
| `visit_time` | Seconds since 2001-01-01 UTC, including a fractional part |

```sql
SELECT v.id AS visit_id,
       i.url,
       v.title,
       v.visit_time AS visit_time_raw,
       datetime(v.visit_time + 978307200, 'unixepoch') AS visited_utc,
       v.origin,
       v.load_successful,
       v.redirect_source,
       v.redirect_destination
FROM history_visits AS v
LEFT JOIN history_items AS i ON i.id = v.history_item
ORDER BY v.visit_time, v.id;
```

The readable timestamp drops subsecond precision; keep the raw value for ordering close events.
A synced visit is not evidence that the page was opened on this Mac. Use device-linked cloud records and local
artifacts to investigate its origin rather than assuming it came from an iPhone.

## Linux: snap and flatpak move the root

!!! info "Skip this unless the host is Linux and the browser is confined"

    Everything here applies only to a browser installed as a snap or a flatpak. On a distribution package 
    (`apt`, `dnf`, `pacman`, or a vendor `.deb`) the paths given in the sections above are correct and 
    this section changes nothing. Jump to the browser family you need.

    Paths below were checked on **Ubuntu 26.04.1 LTS with snap 2.76.3**, and on **Debian 13.6 with flatpak 1.16.6**.

Since Ubuntu 22.04, `apt install firefox` and `apt install chromium-browser` install no browser at all. Both are
transitional packages that depend on `snapd` and pull the snap instead. The user typed an apt command, so the profile
is not where apt would have put it, and `~/.mozilla/firefox` is either absent or stale.

Snap and flatpak each confine the browser to a private home, so nothing lands in `~/.config` or `~/.cache`.

### Snap

snapd points `HOME` at `~/snap/<name>/<revision>` and exposes `~/snap/<name>/common` as storage shared across
revisions.

| Browser | Profile | Cache |
| --- | --- | --- |
| Firefox | `~/snap/firefox/common/.mozilla/firefox/<profile>` | `~/snap/firefox/common/.cache/mozilla/firefox/<profile>` |
| Chromium | `~/snap/chromium/common/chromium/<Profile>` | `~/snap/chromium/common/chromium/<Profile>/Cache` |

Note where Chromium puts its cache: **inside** the profile, not under a separate cache root. That is the fallback
described under [The Chromium cache](cache.md#the-chromium-cache). snapd sets no `XDG_CACHE_HOME`, and the
profile sits outside `$XDG_CONFIG_HOME`, so the mirror into a cache root cannot be computed and Chromium keeps the
cache where the profile is.

Two variants to recognize, both from snapd itself. The experimental `hidden-snap-folder` option moves the whole tree
from `~/snap/` to `~/.snap/data/<name>/`, and a snap migrated to the exposed-home layout gets `HOME=~/Snap/<name>`
(capital S) with `XDG_CACHE_HOME` at `~/snap/<name>/<revision>/xdg-cache`. The three directory names are fixed in
[`dirs/dirs.go`](https://github.com/canonical/snapd/blob/master/dirs/dirs.go) as `UserHomeSnapDir`,
`HiddenSnapDataHomeDir` and `ExposedSnapHomeDir`, and the environment each layout produces is built in `userEnv()` in
[`snap/snapenv/snapenv.go`](https://github.com/canonical/snapd/blob/master/snap/snapenv/snapenv.go).

### Flatpak

Flatpak rewrites `XDG_DATA_HOME`, `XDG_CONFIG_HOME` and `XDG_CACHE_HOME` to three sibling directories under
`~/.var/app/<app id>/`, in `flatpak_context_apply_env_appid()` in
[`common/flatpak-context.c`](https://github.com/flatpak/flatpak/blob/main/common/flatpak-context.c). The usual Linux
layout therefore survives one level down. Confirmed from inside the sandbox:

```console
$ flatpak run --command=sh org.mozilla.firefox -c 'env | grep XDG'
XDG_DATA_HOME=/home/fx5/.var/app/org.mozilla.firefox/data
XDG_CONFIG_HOME=/home/fx5/.var/app/org.mozilla.firefox/config
XDG_CACHE_HOME=/home/fx5/.var/app/org.mozilla.firefox/cache
XDG_STATE_HOME=/home/fx5/.var/app/org.mozilla.firefox/.local/state
```

Desktop session variables from the same output are left out. `XDG_STATE_HOME` is the odd one, under `.local/state`
instead of beside the other three, which the flatpak source flags as deliberate.

| Browser | Profile | Cache |
| --- | --- | --- |
| Chrome | `~/.var/app/com.google.Chrome/config/google-chrome/<profile>` | `~/.var/app/com.google.Chrome/cache/google-chrome/<profile>` |
| Chromium | `~/.var/app/org.chromium.Chromium/config/chromium/<profile>` | `~/.var/app/org.chromium.Chromium/cache/chromium/<profile>` |
| Edge | `~/.var/app/com.microsoft.Edge/config/microsoft-edge/<profile>` | `~/.var/app/com.microsoft.Edge/cache/microsoft-edge/<profile>` |
| Brave | `~/.var/app/com.brave.Browser/config/BraveSoftware/Brave-Browser/<profile>` | `~/.var/app/com.brave.Browser/cache/BraveSoftware/Brave-Browser/<profile>` |
| Opera | `~/.var/app/com.opera.Opera/config/opera/<profile>` | `~/.var/app/com.opera.Opera/cache/opera/<profile>` |
| Firefox | `~/.var/app/org.mozilla.firefox/mozilla/firefox/<profile>` | `~/.var/app/org.mozilla.firefox/cache/mozilla/firefox/<profile>` |

---

<small>
**Primary sources** ·
[Chromium user data directory](https://chromium.googlesource.com/chromium/src/+/main/docs/user_data_dir.md) ·
[Firefox profiles](https://firefox-source-docs.mozilla.org/toolkit/profile/index.html)
</small>

<small>
**Confined Linux packaging**, from the snapd and flatpak sources ·
[`dirs/dirs.go`](https://github.com/canonical/snapd/blob/master/dirs/dirs.go) and
[`snap/snapenv/snapenv.go`](https://github.com/canonical/snapd/blob/master/snap/snapenv/snapenv.go) for the snap
layouts and the environment each one produces ·
[`common/flatpak-context.c`](https://github.com/flatpak/flatpak/blob/main/common/flatpak-context.c) and
[`common/flatpak-run.c`](https://github.com/flatpak/flatpak/blob/main/common/flatpak-run.c) for the XDG redirection and
the per-application data directory ·
[User versus system installs](https://docs.flathub.org/docs/for-users/user-vs-system-install) ·
[`chrome_paths_linux.cc`](https://chromium.googlesource.com/chromium/src/+/main/chrome/common/chrome_paths_linux.cc)
for the cache fallback
</small>

<small>
**Windows paths for Brave, Opera and legacy Edge** ·
*Windows Third-Party Apps Forensics Reference Guide*, DFPS_Windows-Apps-v1.5_0624, by Mattia Epifani with the SANS DFIR
Faculty, © 2024 SANS Institute, from [dfir.sans.org](https://dfir.sans.org/)
</small>

<small>
**Further reading** ·
[Chromium-based Edge from a forensic point of view](https://www.forensicfocus.com/articles/chromium-based-microsoft-edge-from-a-forensic-point-of-view) ·
[Chrome evolution](https://dfir.blog/chrome-evolution) ·
[Opera](https://kb.digital-detective.net/display/BF/Opera)
</small>

---
title: "FCSC 2025: iForensics - iDevice"
description: Identify the iPhone model and iOS build.
date: 2026-09-21
tags:
  - forensics
  - fcsc
---

# iForensics - iDevice

## Challenge

Find the phone's iOS version and model identifier.

The flag format is `FCSC{<model identifier>|<build number>}`. For example, an iPhone 14 Pro Max running iOS 18.4 (22E240) would give `FCSC{iPhone15,3|22E240}`.

**Difficulty:** ⭐

[Official challenge on Hackropole](https://hackropole.fr/en/challenges/forensics/fcsc2025-forensics-iforensics-2/).

## Solution

I used [iLEAPP](https://github.com/abrignoni/iLEAPP) to generate an HTML report from the supplied backup. This also makes the later challenges easier to analyze.

First, extract `backup.tar.xz`:

```sh
λ mkdir -p backup_extracted
λ tar -xf backup.tar.xz -C backup_extracted
```

Supported input types include:

| Type | Description |
| --- | --- |
| `fs` | Folder of extracted files with normal paths and names |
| `zip` | ZIP archive containing files with normal names |
| `tar` | TAR archive |
| `gz` | GZIP-compressed TAR archive |
| `itunes` | iTunes/Finder backup folder with hashed paths and names |
| `file` | Single file |

These types are defined in the [iLEAPP command-line parser](https://github.com/abrignoni/iLEAPP/blob/master/ileapp.py). Here, select `itunes`, point iLEAPP at the extracted backup folder, and generate the report.

The following information comes from `Info.plist` at the backup root:

| Property | Value |
| --- | --- |
| Build Version | 20A362 |
| Device Name | Robert’s iPhone |
| Display Name | Robert’s iPhone |
| GUID | 38DEAAEFE1F58DD8592A16115553392D |
| IMEI | 353846103432059 |
| iTunes Version | 12.10.5 |
| Last Backup Date | 2025-04-07 15:03:33 |
| MEID | 35384610343205 |
| Product Type | iPhone12,3 |
| Product Version | 16.0 |
| Serial Number | C39ZL6V1N6Y6 |
| Target Identifier | 00008030-000E11000A84802E |
| Target Type | Device |
| Unique Identifier | 00008030-000E11000A84802E |

To inspect the same values manually, run `grep` from the backup root:

```sh
λ grep -ri "version" -C2 Info.plist
...<SNIP>...
        <key>Build Version</key>
        <string>20A362</string>
        <key>Device Name</key>
--
        <key>Product Type</key>
        <string>iPhone12,3</string>
        <key>Product Version</key>
        <string>16.0</string>
        <key>Serial Number</key>
--
        <key>iTunes Settings</key>
        <dict/>
        <key>iTunes Version</key>
        <string>12.10.5</string>
</dict>
```

The model identifier is `iPhone12,3`, and iOS 16.0 has build number `20A362` in this backup.

**Flag:** `FCSC{iPhone12,3|20A362}`

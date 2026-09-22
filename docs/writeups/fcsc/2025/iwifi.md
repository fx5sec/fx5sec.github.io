---
title: "FCSC 2025: iForensics - iWiFi"
description: Recover the Wi-Fi network and iCloud account.
date: 2026-09-21
tags:
  - forensics
  - fcsc
---

# iForensics - iWiFi

## Challenge

Find the SSID and BSSID of the Wi-Fi network connected to the phone, along with its associated iCloud account.

The flag format is `FCSC{<SSID>|<BSSID>|<iCloud account>}`. For example: `FCSC{example|00:11:22:33:44:55|example@example.com}`.

**Difficulty:** ⭐

[Official challenge on Hackropole](https://hackropole.fr/en/challenges/forensics/fcsc2025-forensics-iforensics-3/).

## Solution

### Wi-Fi network

I returned to the iLEAPP report. The SSIDs and BSSIDs come from `private/var/preferences/com.apple.wifi.known-networks.plist`:

| SSID | BSSID | Channel Flags | Channel | Last Associated/Roamed At |
| --- | --- | --- | --- | --- |
| FCSC | 42:79:27:3f:3c:93 | 10 | 6 | 2025-04-07 12:25:42+00:00 |
| FCSC | 66:20:95:6c:9b:37 | 10 | 11 | 2025-04-07 14:47:23+00:00 |
| FCSC | e6:3f:c0:1a:40:dd | 10 | 6 | 2025-04-07 12:50:35+00:00 |

The location fields are empty for all three entries. The most recent recorded association is at `2025-04-07 14:47:23+00:00`, giving the first part of the flag: `FCSC{FCSC|66:20:95:6c:9b:37|<...>}`.

### Extract the backup

In an iTunes backup, files are stored under SHA-1 names derived from their domain and relative path. `Manifest.db`, a SQLite database at the backup root, maps those names to the original paths. This naming scheme is described in [LeminLimez's backup research](https://gist.github.com/leminlimez/c602c067349140fe979410ef69d39c28/01a260d9dc8420d059594040e6807fd467836679).

To avoid looking up each file manually, I used [ibackupextractor](https://github.com/unixzii/ibackupextractor) to restore readable paths:

```sh
λ ibackupextractor extract --all backup_extracted/ backup_lisible

λ ll backup_lisible
total 1.2M
drwxrwxr-x 19 fx5 fx5 4.0K Sep 17 22:53 .
drwxrwxr-x  6 fx5 fx5 4.0K Sep 17 22:52 ..
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 Breakpad
drwxrwxr-x  3 fx5 fx5 4.0K Sep 17 22:53 com.apple.MobileSMS
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 com.apple.xpc.launchd
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 CRDTModelFileSync
drwxrwxr-x  6 fx5 fx5 4.0K Sep 17 22:53 Documents
drwxrwxr-x  3 fx5 fx5 4.0K Sep 17 22:53 fba_extentions_event_store
drwxrwxr-x  5 fx5 fx5 4.0K Sep 17 22:53 Health
drwxrwxr-x  3 fx5 fx5 4.0K Sep 17 22:53 IABPCMAdConversionsStorage.IABPCMControllerCache
drwxrwxr-x 47 fx5 fx5 4.0K Sep 17 22:53 Library
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 locationd
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 Maps
drwxrwxr-x  4 fx5 fx5 4.0K Sep 17 22:53 Media
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 mobile
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 ProvisioningProfiles
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 ReinstallDetection
drwxrwxr-x  2 fx5 fx5 4.0K Sep 17 22:53 SystemConfiguration
drwxrwxr-x  3 fx5 fx5 4.0K Sep 17 22:53 trustd
-rw-r--r--  1 fx5 fx5  286 Sep 17 22:53 BDSICloudIdentityToken.plist
-rw-r--r--  1 fx5 fx5  353 Sep 17 22:53 com.apple.networkextension.control.plist
-rw-r--r--  1 fx5 fx5  14K Sep 17 22:53 com.apple.networkextension.plist
-rw-r--r--  1 fx5 fx5    0 Sep 17 22:53 com.apple.notes.databaseopen.lock
-rw-r--r--  1 fx5 fx5 1.1K Sep 17 22:53 com.apple.wifi.known-networks.plist
-rw-r--r--  1 fx5 fx5  82K Sep 17 22:53 config.plist
-rw-r--r--  1 fx5 fx5  223 Sep 17 22:53 .FirstUnlock
-rw-r--r--  1 fx5 fx5 293K Sep 17 22:53 keychain-backup.plist
-rw-r--r--  1 fx5 fx5 420K Sep 17 22:53 linkd.metadatastore.sqlite3
-rw-r--r--  1 fx5 fx5 284K Sep 17 22:53 NoteStore.sqlite
-rw-r--r--  1 fx5 fx5  166 Sep 17 22:53 past-sessions.json
-rw-r--r--  1 fx5 fx5 2.2K Sep 17 22:53 PlugInKit-Annotations
```

### iCloud account

Account information is stored in `/private/var/mobile/Library/Accounts/Accounts3.sqlite`. After extraction, I found it at `backup_lisible/Library/Accounts/VerifiedBackup/Accounts3.sqlite`.

The `ZACCOUNT` table contains this iCloud entry:

| Field | Value |
| --- | --- |
| ZACCOUNTDESCRIPTION | iCloud |
| ZIDENTIFIER | 89695A59-F596-4E3A-952F-034A30421C79 |
| ZMODIFICATIONID | 2E60F588-BF6E-418B-8D2E-DE6808A64FDD |
| Unlabeled field in the export | com.apple.purplebuddy |
| ZUSERNAME | robertswigert@icloud.com |

**Flag:** `FCSC{FCSC|66:20:95:6c:9b:37|robertswigert@icloud.com}`

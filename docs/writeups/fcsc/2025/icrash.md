---
title: "FCSC 2025: iForensics - iCrash"
description: Find a flag among the iPhone crash logs.
date: 2026-09-21
tags:
  - forensics
  - fcsc
---

# iForensics - iCrash

## Challenge

At customs, an officer asks for your phone and its unlock code. The phone is returned a few hours later.

Suspicious, you send it to ANSSI's CERT-FR for analysis. The analysts collect a *sysdiagnose* and a *backup*.

A flag seems to be hidden where the phone stores crash logs.

**Difficulty:** Intro

[Official challenge on Hackropole](https://hackropole.fr/en/challenges/forensics/fcsc2025-forensics-iforensics-1/).

## Solution

The crash logs are under `/private/var/mobile/Library/Logs/CrashReporter/`. A recursive search finds the flag:

```sh
λ grep -iR "fcsc" sysdiagnose_and_crashes/private
sysdiagnose_and_crashes/private/var/mobile/Library/Logs/CrashReporter/fcsc_intro.txt:FCSC{7a1ca2d4f17d4e1aa8936f2e906f0be8}
```

**Flag:** `FCSC{7a1ca2d4f17d4e1aa8936f2e906f0be8}`

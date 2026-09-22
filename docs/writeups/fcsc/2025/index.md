---
title: FCSC 2025
description: Investigate an iPhone through nine forensic challenges.
---

# FCSC 2025

The **iForensics** series follows an iPhone returned after a customs inspection.
Its owner suspects tampering. A backup and a sysdiagnose archive provide the
evidence for nine challenges, from device identification to the initial infection.

Challenge descriptions and evidence are available on
[Hackropole](https://hackropole.fr/en/challenges/forensics/fcsc2025-forensics-iforensics-7/).
These write-ups follow my investigation, including the useful pivots and dead ends.

## iForensics

The challenges are independent, except **iBackdoor 2/2**, which builds on
**iBackdoor 1&#47;2**. Reading them in order follows the investigation.

| Challenge | Difficulty | Investigation |
| --- | --- | --- |
| [iCrash](icrash.md) | Intro | Locate a flag among the crash reports. |
| [iDevice](idevice.md) | ★ | Identify the device model and iOS build. |
| [iWiFi](iwifi.md) | ★ | Recover the Wi-Fi network and iCloud account. |
| [iTreasure](itreasure.md) | ★ | Retrieve an image sent through Messages. |
| [iNvisible](invisible.md) | ★★ | Identify the recipient of an unsent message. |
| [iBackdoor 1&#47;2](ibackdoor-1.md) | ★★ | Find the compromised app and suspicious processes. |
| [iBackdoor 2/2](ibackdoor-2.md) | ★★ | Trace the legitimate app's extraction and removal. |
| [iC2](ic2.md) | ★★★ | Identify the implant and its C2 endpoint. |
| [iCompromise](icompromise.md) | ★★★ | Reconstruct the initial infection vector and time. |

The collection uses **iLEAPP**, **ibackupextractor**, the **Sysdiagnose Analysis
Framework**, SQL, and log analysis. Commands and paths refer to the supplied
challenge artifacts.

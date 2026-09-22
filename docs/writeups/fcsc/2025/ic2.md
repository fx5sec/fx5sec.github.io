---
title: "FCSC 2025: iForensics - iC2"
description: Identify the implant and its C2 endpoint.
date: 2026-09-21
tags:
  - forensics
  - fcsc
---

# iForensics - iC2

## Challenge

Find the name of the malicious tool deployed on the phone, as well as the protocol, IP address, and communication port of its C2 server.

The flag is in the format `FCSC{<tool>|<protocol>|<IP address>|<port>}`. For example, if the tool is `Cobalt Strike`, the protocol is `TCP`, the IP address is `127.0.0.1`, and the port is `1337`: `FCSC{Cobalt Strike|TCP|127.0.0.1|1337}`.

**Difficulty:** ⭐⭐⭐

[Original challenge on Hackropole](https://hackropole.fr/en/challenges/forensics/fcsc2025-forensics-iforensics-8/).

## Solution

### Decode the C2 endpoint

In [iBackdoor 1](ibackdoor-1.md), we identified the backdoor and created `ps_processed.txt`. A Base64 argument is not exactly subtle:

```sh
λ cat ps_processed.txt | grep -i "Signal.app"
2025-04-07T08:06:18.000000-07:00        279     /var/containers/Bundle/Application/4B6E715E-641B-4F43-B39B-CA9AE3E8B73B/Signal.app/mussel dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==
2025-04-07T08:06:18.000000-07:00        330     /var/containers/Bundle/Application/4B6E715E-641B-4F43-B39B-CA9AE3E8B73B/Signal.app/mussel dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==
2025-04-07T08:06:18.000000-07:00        344     /var/containers/Bundle/Application/4B6E715E-641B-4F43-B39B-CA9AE3E8B73B/Signal.app/Signal
2025-04-07T08:06:18.000000-07:00        345     /var/containers/Bundle/Application/4B6E715E-641B-4F43-B39B-CA9AE3E8B73B/Signal.app/mussel dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==

λ echo "dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==" | base64 -d
tcp://98.66.154.235:29552
```

That gives us three of the four flag components:

- Protocol: `TCP`
- IP address: `98.66.154.235`
- Port: `29552`

### Identify the tool

The remaining step is to identify the tool. Implant names can be arbitrary or imitate legitimate processes. Here, the name `mussel` offers a useful search term.

Searching for `"mussel" ios implant` led me to the developer's [SeaShell article](https://web.archive.org/web/20250117023041/https://blog.entysec.com/posts/seashell-ios-malware/). It explicitly names `mussel` as the executable SeaShell adds to an application bundle and describes using TrollStore to install the modified application. This matches the evidence from the previous challenges.

```text
FCSC{SeaShell|TCP|98.66.154.235|29552}
```

After the earlier challenges, this one felt easier than its three-star rating suggested.

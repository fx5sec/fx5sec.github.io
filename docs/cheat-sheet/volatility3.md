---
title: Volatility 3
description: Volatility 3 field reference
tags:
  - forensics
  - cheatsheet
---

# Volatility 3

!!! info "Verified against"

    Volatility 3 **2.28.0** on Python 3.13, checked on 2026-08-29 by querying the
    installed framework directly rather than from documentation. Plugin names move
    between releases. Run `vol -h` against your own install before trusting a name
    on this page.

## Invocation

```
vol [framework options] PLUGIN [plugin options]
```

Both the short module path and the full class path work, so `windows.pslist` and
`windows.pslist.PsList` are equivalent. The short form is used throughout this page.

| Option | Purpose |
| --- | --- |
| `-f FILE` | The memory image. Shorthand for `--single-location=file://` |
| `-o DIR` | Where dumped files are written |
| `-r RENDERER` | `quick`, `none`, `csv`, `pretty`, `json`, `jsonl`, `arrow`, `parquet` |
| `--filters` | Filter rows, in the form `[+-]columnname,pattern[!]` |
| `--hide-columns` | Space separated column prefixes to drop from the output |
| `-s DIRS` | Extra symbol table directories, semicolon separated |
| `-p DIRS` | Extra plugin directories, semicolon separated |
| `--offline` | Never reach out for remote ISF files |
| `--parallelism` | `processes`, `threads` or `off` |
| `--clear-cache` | Wipe the cache when a stale symbol table is suspected |
| `-v` | Repeatable. `-vvv` is what you want when a plugin fails silently |

## Deprecated names in 2.28

Twenty plugins are now marked deprecated in favour of dedicated namespaces. The old
names still resolve, so nothing breaks, but a sheet written before 2.28 will point
you at the wrong half of the framework.

| Deprecated | Use instead |
| --- | --- |
| `windows.malfind` | `windows.malware.malfind` |
| `windows.psxview` | `windows.malware.psxview` |
| `windows.hollowprocesses` | `windows.malware.hollowprocesses` |
| `windows.processghosting` | `windows.malware.processghosting` |
| `windows.suspicious_threads` | `windows.malware.suspicious_threads` |
| `windows.unhooked_system_calls` | `windows.malware.unhooked_system_calls` |
| `windows.drivermodule` | `windows.malware.drivermodule` |
| `windows.svcdiff` | `windows.malware.svcdiff` |
| `windows.amcache` | `windows.registry.amcache` |
| `windows.scheduled_tasks` | `windows.registry.scheduled_tasks` |
| `linux.malfind` | `linux.malware.malfind` |
| `linux.check_afinfo` | `linux.malware.check_afinfo` |
| `linux.check_creds` | `linux.malware.check_creds` |
| `linux.check_idt` | `linux.malware.check_idt` |
| `linux.check_modules` | `linux.malware.check_modules` |
| `linux.check_syscall` | `linux.malware.check_syscall` |
| `linux.hidden_modules` | `linux.malware.hidden_modules` |
| `linux.keyboard_notifiers` | `linux.malware.keyboard_notifiers` |
| `linux.modxview` | `linux.malware.modxview` |
| `linux.netfilter` | `linux.malware.netfilter` |
| `linux.tty_check` | `linux.malware.tty_check` |

## Triage order

Establish the ground truth before hunting. Each step should change what you look at
next, otherwise it is wasted.

1. `windows.info` confirms the profile, the kernel build and the acquisition time.
   If this fails, nothing downstream is trustworthy.
2. `windows.pstree` gives you the parent and child relationships in one pass. Odd
   parentage is visible here and invisible in a flat list.
3. `windows.cmdline` puts arguments next to the process names. Half of the
   suspicious processes give themselves away in their own command line.
4. `windows.netscan` maps connections to owning processes, including sockets that
   are already closed.
5. `windows.malware.malfind` looks for executable private memory with no backing
   file on disk.
6. `windows.malware.psxview` cross-checks four process enumeration methods against
   each other. Disagreement between them is the finding.

## Windows plugins by question

### Processes

| Plugin | Answers |
| --- | --- |
| `windows.pslist` | The linked list of active processes. `--pid`, `--dump` |
| `windows.psscan` | Pool scan. Finds terminated and unlinked processes `pslist` misses |
| `windows.pstree` | The same data as a parent and child hierarchy |
| `windows.malware.psxview` | Which enumeration methods disagree about a process |
| `windows.cmdline` | Command line arguments per process |
| `windows.envars` | Environment variables per process |
| `windows.sessions` | Which session and logged-on user a process belongs to |
| `windows.privileges` | Privileges held and enabled per process |
| `windows.getsids` | The SIDs a process is running under |
| `windows.handles` | Open handles: files, keys, mutexes, sections |
| `windows.joblinks` | Job object membership, useful around container-like isolation |

### Injected and hidden code

| Plugin | Answers |
| --- | --- |
| `windows.malware.malfind` | Executable private memory with no file backing it |
| `windows.malware.hollowprocesses` | Processes whose image no longer matches its file |
| `windows.malware.processghosting` | Processes whose backing file was deleted before execution |
| `windows.malware.ldrmodules` | Modules missing from one of the three PEB lists |
| `windows.malware.pebmasquerade` | A PEB claiming a different image path than the real one |
| `windows.malware.suspicious_threads` | Threads starting outside any mapped module |
| `windows.malware.unhooked_system_calls` | `ntdll` stubs that no longer match the on-disk code |
| `windows.vadinfo` | The VAD tree, with protections. `--dump` extracts a region |
| `windows.threads` | Thread enumeration, with start addresses |

### Modules, drivers and services

| Plugin | Answers |
| --- | --- |
| `windows.modules` | Loaded kernel modules from the linked list |
| `windows.modscan` | Pool scan for modules, including unlinked ones |
| `windows.malware.drivermodule` | Drivers hidden by a rootkit |
| `windows.driverirp` | IRP handler table, to spot hooks |
| `windows.ssdt` | System service descriptor table entries |
| `windows.callbacks` | Registered kernel callbacks |
| `windows.svcscan` | Services, including ones not in the service list |
| `windows.malware.svcdiff` | Services found by scanning but absent from the list |
| `windows.unloadedmodules` | Recently unloaded drivers, a common anti-forensics trace |

### Network

| Plugin | Answers |
| --- | --- |
| `windows.netscan` | Connections and listeners, with owning process |
| `windows.netstat` | The same via the network module structures |

`--include-corrupt` on `netscan` widens the results by relaxing validation. It
raises the false positive rate, so treat what it surfaces as leads, not facts.

### Registry

| Plugin | Answers |
| --- | --- |
| `windows.registry.hivelist` | Loaded hives and their offsets. `--dump` extracts them |
| `windows.registry.hivescan` | Pool scan for hives that are not in the list |
| `windows.registry.printkey` | Key contents. `--key`, `--recurse`, `--offset` |
| `windows.registry.userassist` | GUI program execution from UserAssist |
| `windows.registry.amcache` | Application execution evidence from AmCache |
| `windows.registry.scheduled_tasks` | Task definitions with triggers and actions |
| `windows.registry.certificates` | Certificates in the registry stores |

### Files and credentials

| Plugin | Answers |
| --- | --- |
| `windows.filescan` | File objects in memory, with offsets to dump from |
| `windows.dumpfiles` | Extract a file. `--pid`, `--virtaddr`, `--physaddr`, `--filter` |
| `windows.mftscan` | MFT records recoverable from memory |
| `windows.hashdump` | Local account hashes from the SAM |
| `windows.lsadump` | LSA secrets |
| `windows.cachedump` | Cached domain credentials |
| `windows.malware.skeleton_key_check` | The skeleton key patch on a domain controller |

### Console history

| Plugin | Answers |
| --- | --- |
| `windows.cmdscan` | Command history structures found by scanning |
| `windows.consoles` | Console buffers, including command output, not just input |

`windows.consoles` is the one to reach for when `cmdscan` returns little. It
recovers what was printed back to the operator, which often carries more than the
commands themselves.

## Linux

| Plugin | Answers |
| --- | --- |
| `linux.pslist`, `linux.psscan`, `linux.pstree` | Process enumeration by three routes |
| `linux.psaux` | Processes with their full argument vectors |
| `linux.pscallstack` | Kernel call stacks per process |
| `linux.bash` | Recoverable bash history from process memory |
| `linux.lsof` | Open file descriptors |
| `linux.sockstat`, `linux.sockscan` | Sockets by list walk and by scan |
| `linux.lsmod` | Loaded kernel modules |
| `linux.malware.hidden_modules` | Modules carved from memory but absent from the list |
| `linux.malware.modxview` | lsmod, check_modules and hidden_modules correlated |
| `linux.malware.check_syscall` | Syscall table hooks |
| `linux.malware.malfind` | Injected code in process memory |
| `linux.elfs` | ELF images mapped into processes |
| `linux.library_list` | Shared libraries per process |
| `linux.mountinfo` | Mounted filesystems |
| `linux.iomem` | Physical memory map |
| `linux.kmsg` | Kernel ring buffer |
| `linux.ebpf` | Loaded eBPF programs |
| `linux.tracing.ftrace` | ftrace hooks, a modern hiding place |
| `linux.pagecache` | Files cached in memory |

## macOS

| Plugin | Answers |
| --- | --- |
| `mac.pslist`, `mac.pstree`, `mac.psaux` | Process enumeration |
| `mac.lsof` | Open files |
| `mac.netstat` | Network connections |
| `mac.socket_filters` | Socket filters, a hooking point |
| `mac.lsmod` | Loaded kernel extensions |
| `mac.malfind` | Injected code |
| `mac.bash` | Recoverable shell history |
| `mac.kauth_listeners`, `mac.kauth_scopes` | Kauth listeners, used by security tools and rootkits alike |
| `mac.trustedbsd` | TrustedBSD policy hooks |
| `mac.vfsevents` | Filesystem event listeners |
| `mac.check_syscall`, `mac.check_sysctl`, `mac.check_trap_table` | Table integrity checks |
| `mac.list_files` | Files referenced in the vnode cache |

## Cross-platform

| Plugin | Answers |
| --- | --- |
| `banners` | Kernel banners found in the image. The starting point when the profile is unknown |
| `isfinfo` | Which symbol tables are available locally |
| `timeliner` | Every timestamp every plugin can produce, as one timeline |
| `yarascan` | YARA rules across the image |
| `regexscan` | Regular expression search across the image |
| `layerwriter` | Write a memory layer out to a file |
| `vmscan` | Locate virtual machine memory inside the image |
| `frameworkinfo` | The framework's own configuration, for bug reports |
| `configwriter` | Dump the resolved configuration as JSON |

## Symbols

Windows symbol tables are resolved automatically against the remote ISF server on
first use, then cached. Linux and macOS have no equivalent: you need an ISF built
from the exact kernel of the acquired system.

```bash
vol -f mem.raw banners                 # identify the kernel
vol -f mem.raw isfinfo                 # what is available locally
vol -s /path/to/symbols -f mem.raw linux.pslist
vol --offline -f mem.raw windows.info  # air-gapped analysis
vol --clear-cache                      # when a stale table is suspected
```

## Output

```bash
vol -f mem.raw -r csv windows.pslist > pslist.csv
vol -f mem.raw -o ./dumped windows.dumpfiles --pid 1234
vol -f mem.raw windows.pslist --filters "+ImageFileName,svchost"
vol -f mem.raw windows.pslist --hide-columns Offset Threads
```

`--filters` takes `[+-]columnname,pattern[!]`, where `+` includes and `-` excludes.
It filters the rendered output, not the analysis, so it saves your eyes rather than
your time.

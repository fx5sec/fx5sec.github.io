---
title: Windows Event IDs
description: Windows event IDs for forensic investigations.
tags:
  - forensics
  - cheatsheet
---
# Windows Event IDs

I wasn't going to reinvent the wheel. This cheat sheet is based on
[13Cubed's Windows Event Log Cheat Sheet](https://cdn.13cubed.com/downloads/windows_event_log_cheat_sheet.pdf),
with additional events and investigation notes, including the full documented
Sysmon event list.

Events are grouped by log channel, with default EVTX paths below each heading.
Always match the provider as well as the event ID: different providers reuse IDs.
Event links point to documentation or recorded event examples where available.
The **Significance** column adds investigation context only where useful; a blank
cell means no extra explanation is needed. These notes suggest leads, not proof
of compromise. Availability depends on Windows version, audit policy, and logging
configuration.

## Security

Location: `%SystemRoot%\System32\winevt\Logs\Security.evtx`

Provider: `Microsoft-Windows-Security-Auditing` (1102: `Microsoft-Windows-Eventlog`).

| Event ID | Description | Significance |
| --- | --- | --- |
| [4608][4608] | Windows startup; the auditing subsystem initialized. | Provides a startup reference point for the timeline. |
| [4624][4624] | An account was successfully logged on. (See Logon Type Codes) |  |
| [4625][4625] | An account failed to log on. |  |
| [4634][4634] | An account was logged off. |  |
| [4647][4647] | User initiated logoff. (In place of 4634 for Interactive and RemoteInteractive logons) |  |
| [4648][4648] | A logon was attempted using explicit credentials. (RunAs) |  |
| [4672][4672] | Special privileges assigned to new logon. (Admin login) |  |
| [4776][4776] | The domain controller attempted to validate the credentials for an account. (DC) |  |
| [4768][4768] | A Kerberos authentication ticket (TGT) was requested. |  |
| [4769][4769] | A Kerberos service ticket was requested. |  |
| [4771][4771] | Kerberos pre-authentication failed. |  |
| [4720][4720] | A user account was created. |  |
| [4728][4728] | A member was added to a security-enabled global group. | For Domain Admins, check the target group SID: the domain SID plus RID `512`. Review the actor and added member on the DC. |
| [4722][4722] | A user account was enabled. |  |
| [4688][4688] | A new process has been created. (If audited; some Windows processes logged by default) |  |
| [4698][4698] | A scheduled task was created. (If audited) |  |
| [4798][4798] | A user's local group membership was enumerated. |  |
| [4799][4799] | A security-enabled local group membership was enumerated. |  |
| [5140][5140] | A network share object was accessed. |  |
| [5145][5145] | A network share object was checked to see whether client can be granted desired access. |  |
| [5379][5379] | Stored credentials were read or enumerated in Credential Manager. | Correlate the account, logon ID, and caller process where available. Routine applications also generate this event. |
| [1102][1102] | The audit log was cleared. (Security) |  |

  [4608]: https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4608
  [4728]: https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/audit-security-group-management
  [5379]: https://learn.microsoft.com/en-gb/answers/questions/1045216/event-5379
  [4624]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624
  [4625]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625
  [4634]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4634
  [4647]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4647
  [4648]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4648
  [4672]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672
  [4776]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4776
  [4768]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768
  [4769]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4769
  [4771]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4771
  [4720]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4720
  [4722]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4722
  [4688]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688
  [4698]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4698
  [4798]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4798
  [4799]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4799
  [5140]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-5140
  [5145]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-5145
  [1102]: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-1102

The [Security Identifiers reference](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers) documents the Domain Admins SID. The 5379 link is a Microsoft Q&A example, not a dedicated audit-event reference.

### Logon Type Codes

| Type | Description | Significance |
| --- | --- | --- |
| 2 | Console |  |
| 3 | Network | Common in lateral movement, but also routine network access. |
| 4 | Batch (Scheduled Tasks) |  |
| 5 | Windows Services |  |
| 7 | Unlock |  |
| 8 | Network (Cleartext Logon) | Credentials reach the authentication package in cleartext; this does not establish cleartext network transport. |
| 9 | Alternate Credentials Specified (RunAs) | RunAs /netonly uses alternate credentials for outbound access. |
| 10 | Remote Interactive (RDP) | Useful for tracking interactive RDP access. |
| 11 | Cached Credentials (e.g., Offline DC) |  |
| 12 | Cached Remote Interactive (RDP, similar to Type 10) |  |
| 13 | Cached Unlock (Similar to Type 7) |  |


Reference: [EID 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624) enumerates the types as they appear in the event, and [`SECURITY_LOGON_TYPE`](https://learn.microsoft.com/en-us/windows/win32/api/ntsecapi/ne-ntsecapi-security_logon_type) is the underlying enumeration. The [administrative tools and logon types reference](https://learn.microsoft.com/en-us/windows-server/identity/securing-privileged-access/reference-tools-logon-types) maps each type to the credentials it leaves behind in LSA, which is the part that matters when you are chasing credential theft.

## System

Location: `%SystemRoot%\System32\winevt\Logs\System.evtx`

| Event ID | Provider | Description | Significance |
| --- | --- | --- | --- |
| 7045 | `Service Control Manager` | A new service was installed in the system. (4697 in Security) | Check for service-based persistence or remote execution, including PsExec. |
| 7034 | `Service Control Manager` | The *x* service terminated unexpectedly. It has done this *y* time(s). | Correlate crashes with nearby process and application events. |
| [7036][7036] | `Service Control Manager` | A service entered a new state, such as running or stopped. | Reconstruct starts and stops, especially security services. A state change alone does not mean an unexpected stop. |
| [7001][7001] | `Microsoft-Windows-Winlogon` | User logon notification for the Customer Experience Improvement Program. | Helps place a user logon on the timeline. Filter by provider to avoid unrelated 7001 service errors. |
| 7009 | `Service Control Manager` | A timeout was reached (*x* milliseconds) while waiting for the *y* service to connect. |  |
| 104 | `Microsoft-Windows-Eventlog` | The *x* log file was cleared. (Will show System, Application, and other logs cleared) | Check which log was cleared and whether the action was expected. |

[7036]: https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/is-group-policy-slowing-me-down/ba-p/259701/
[7001]: https://techcommunity.microsoft.com/discussions/windowspowershell/powershell-ile-rdp-logon-logoff-kayitlarini-g%C3%B6r%C3%BCnt%C3%BCleme/1376308/

## Application

Location: `%SystemRoot%\System32\winevt\Logs\Application.evtx`

| Event ID | Provider | Description | Significance |
| --- | --- | --- | --- |
| *1000 | `Application Error` | Application Error | Crashes near suspected exploitation can help establish a timeline. |
| [1040][1040] | `MsiInstaller` | A Windows Installer transaction began. | Identifies the package or product and client process ID. Does not establish successful installation. |
| [11707][11707] | `MsiInstaller` | An MSI product installation completed successfully. | Correlate with 1040 to establish the installation timeline. |
| *1002 | `Application Hang` | Application Hang |  |

*\*Remember, third-party software (like Antivirus) can also write to this log!*

[1040]: https://learn.microsoft.com/en-us/answers/questions/1483726/what-is-addinutil-exe-why-is-it-running
[11707]: https://learn.microsoft.com/en-us/windows/win32/msi/event-logging

The 1040 reference contains an observed event example from Microsoft Q&A.

### Application (ESENT Provider)

Location: `%SystemRoot%\System32\winevt\Logs\Application.evtx`

| Event ID | Description | Significance |
| --- | --- | --- |
| 216 | A database location change was detected. | An unexpected ntds.dit path can warrant investigating a database copy. |
| 325 | The database engine created a new database. | A new database involving ntds.dit warrants checking for credential theft. |
| 326 | The database engine attached a database. |  |
| 327 | The database engine detached a database. |  |


Reference: [Extensible Storage Engine](https://learn.microsoft.com/en-us/windows/win32/extensible-storage-engine/extensible-storage-engine).

## Windows-PowerShell

Location: `%SystemRoot%\System32\winevt\Logs\Windows PowerShell.evtx`

| Event ID | Description | Significance |
| --- | --- | --- |
| 400 | Engine state is changed from None to Available. | Use the host application and timing to investigate unexpected PowerShell execution. |
| 600 | Provider "x" is Started. |  |


## Microsoft-Windows-PowerShell/Operational

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-PowerShell%4Operational.evtx`

| Event ID | Description | Significance |
| --- | --- | --- |
| *4104 | 4104, Creating Scriptblock text (1 of 1): (Scriptblock Logging) | Inspect script content, including code revealed during execution. |

*\*Enabled by default in PowerShell v5 and later for scripts identified as potentially malicious, logged as warnings*


Reference: [about_Logging_Windows](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows).

## Microsoft-Windows-Sysmon/Operational

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational.evtx`

Provider: `Microsoft-Windows-Sysmon`.

All IDs documented in the [Sysmon reference][sysmon] as of 2026-09-07 are
included below: 1-29 and 255. Collection depends on the installed version and
configuration; network and module logging are disabled by default.

| Event ID | Description | Significance |
| --- | --- | --- |
| [1][sysmon] | Process started. | Correlate execution with its parent and command line. |
| [2][sysmon] | File creation time changed. | Check for timestomping. |
| [3][sysmon] | Network connection. | Link endpoints to a process. |
| [4][sysmon] | Sysmon started or stopped. | Investigate unexpected collection gaps. |
| [5][sysmon] | Process exited. |  |
| [6][sysmon] | Driver loaded. | Review unfamiliar drivers. |
| [7][sysmon] | Module loaded. | Check suspicious DLL paths. |
| [8][sysmon] | Remote thread created. | Potential process injection. |
| [9][sysmon] | Raw disk read. |  |
| [10][sysmon] | Another process accessed. | Review unexpected LSASS access. |
| [11][sysmon] | File created or overwritten. |  |
| [12][sysmon] | Registry object created or deleted. |  |
| [13][sysmon] | Registry value set. | Check persistence locations. |
| [14][sysmon] | Registry object renamed. |  |
| [15][sysmon] | Named stream created. | Inspect alternate data streams. |
| [16][sysmon] | Sysmon configuration changed. | Review changes affecting visibility. |
| [17][sysmon] | Named pipe created. |  |
| [18][sysmon] | Named pipe connected. |  |
| [19][sysmon] | WMI filter registered. | Correlate 19-21 for persistence. |
| [20][sysmon] | WMI consumer registered. |  |
| [21][sysmon] | WMI consumer bound to filter. |  |
| [22][sysmon] | DNS query. |  |
| [23][sysmon] | File deleted and archived. | Retrieve the archived artifact. |
| [24][sysmon] | Clipboard changed. |  |
| [25][sysmon] | Process image tampered with. | Investigate process hollowing. |
| [26][sysmon] | File deletion logged, without archival. |  |
| [27][sysmon] | PE creation blocked. |  |
| [28][sysmon] | File shredding blocked. |  |
| [29][sysmon] | PE creation detected. |  |
| [255][sysmon] | Sysmon error. | Check telemetry completeness. |

[sysmon]: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon#events

## Microsoft-Windows-TaskScheduler/Operational

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-TaskScheduler%4Operational.evtx`

| Event ID | Description | Significance |
| --- | --- | --- |
| 106 | The user *x* registered the Task Scheduler task *y*. (New Scheduled Task) | Check the task action and creator for persistence. |
| 141 | User *x* deleted Task Scheduler task *y*. | May help identify cleanup after task execution. |
| 100 | Task Scheduler started the *x* instance of the *y* task for user *z*. | Establishes when a registered task actually ran. |
| 102 | Task Scheduler successfully finished the *x* instance of the *y* task for user *z*. |  |


## Microsoft-Windows-Windows Defender/Operational

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-Windows Defender%4Operational.evtx`

| Event ID | Description | Significance |
| --- | --- | --- |
| 1116 | The antimalware platform detected malware or other potentially unwanted software. | Check the matching 1117 action and its outcome. |
| [5001][5001] | Microsoft Defender Antivirus real-time protection was disabled. | Check for an expected configuration change or possible defense evasion. |
| 1117 | The antimalware platform performed an action to protect your system from malware or other potentially unwanted software. |  |


Reference: [Microsoft Defender Antivirus event IDs and error codes](https://learn.microsoft.com/en-us/defender-endpoint/troubleshoot-microsoft-defender-antivirus), which documents 1116, 1117, and 5001 individually.

[5001]: https://learn.microsoft.com/en-us/defender-endpoint/troubleshoot-microsoft-defender-antivirus#event-id-5001

## Microsoft-Windows-Windows Firewall With Advanced Security/Firewall

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-Windows Firewall With Advanced Security%4Firewall.evtx`

Provider: `Microsoft-Windows-Windows Firewall With Advanced Security`.

| Event ID | Description | Significance |
| --- | --- | --- |
| [2004][2004] | A rule was added to the Windows Firewall exception list. | Check the application, direction, action, ports, and scope for newly allowed access. |

[2004]: https://learn.microsoft.com/en-us/answers/questions/3145988/%28solved%29-windows-firewall-exception-list-%28-%29-is-th

Reference: the linked Microsoft Q&A post includes the original 2004 event and
its channel, provider, and rule fields.

## Microsoft-Windows-DNS-Client/Operational

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-DNS-Client%4Operational.evtx`

Provider: `Microsoft-Windows-DNS-Client`.

| Event ID | Description | Significance |
| --- | --- | --- |
| [3006][3006] | A DNS query was initiated. | Establishes the requested name and query type, but does not prove a network request or successful resolution. |
| [3018][3018] | A local DNS cache lookup returned a result and status. | For cached A or AAAA answers, `QueryResults` can recover the IPv4 or IPv6 address associated with the name. |

For 3018, a cache miss can leave `QueryResults` empty. Check `Status` and
`QueryType` too: an empty result alone is not proof that the name does not exist.
A cached answer does not establish that the client connected to the address.

References: [NXLog's collected DNS Client events][3006] show the channel and
3006 message; [Secureworks' collected 3018 events][3018] show the cache lookup
message and an empty `QueryResults` example.

[3006]: https://docs.nxlog.co/integrations/dns/dns-monitoring-windows.html#collect-native-dns-client-logs
[3018]: https://docs.taegis.secureworks.com/integration/connectEndpoint/red_cloak_endpoint_agent_technical_details/

## Microsoft-Windows-TerminalServices-LocalSessionManager/Operational

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-TerminalServices-LocalSessionManager%4Operational.evtx`

| Event ID | Description | Significance |
| --- | --- | --- |
| 21 | Remote Desktop Services: Session logon succeeded: | Correlate session and source address with Security logons. |
| 22 | Remote Desktop Services: Shell start notification received: |  |
| 23 | Remote Desktop Services: Session logoff succeeded: |  |
| 24 | Remote Desktop Services: Session has been disconnected: | A disconnected session can remain logged on. |
| 25 | Remote Desktop Services: Session reconnection succeeded: | Links activity to a previously disconnected session. |


Reference: [Remote Desktop Services](https://learn.microsoft.com/en-us/windows/win32/termserv/terminal-services-portal), for this channel and the two RDS channels below. Microsoft does not document any of their operational event IDs individually, which is why 1149 and 1029 carry the notes they do.

## Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-TerminalServices-RemoteConnectionManager%4Operational.evtx`

| Event ID | Description | Significance |
| --- | --- | --- |
| *1149 | Remote Desktop Services: User authentication succeeded: | Use the user and source address to correlate subsequent session events. |
| 261 | Listener RDP-Tcp received a connection |  |

*\*Event ID 1149 indicates successful network authentication, which occurs prior to user authentication, but in newer versions of Windows it has been observed that this event is only logged when the subsequent user authentication is successful*


## Microsoft-Windows-TerminalServices-RDPClient/Operational

Location: `%SystemRoot%\System32\winevt\Logs\Microsoft-Windows-TerminalServices-RDPClient%4Operational.evtx`

| Event ID | Description | Significance |
| --- | --- | --- |
| *1029 | `Base64(SHA256(UserName)) is = HASH` | Evidence of outbound RDP; compare hashes of candidate usernames. |


For the code and more explanation, see the [article](https://nullsec.us/windows-event-id-1029-hashes/) from nullsec.

```python title="Hash Generator"
import hashlib,base64
username = "Administrator"
username = username.decode('utf-8').encode('utf-16le')
hash = hashlib.sha256(username).digest() # note NOT .hexdigest()
print base64.b64encode(hash)
```

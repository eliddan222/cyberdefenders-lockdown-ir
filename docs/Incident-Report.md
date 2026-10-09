# Incident Response Report — INC-2026-001

## Executive Summary
TechNova System's SOC detected suspicious outbound traffic from the organization's public-facing IIS server, that was suggestive of web-shell connections to an unknown host. The SOC analyst discovered three artifacts and presented them to the forensic examiner for investigation. These artifacts were a packet capture, a memory image, and a recovered executable. Analysis confirmed that the IIS server was compromised during a string of events beginning with automated reconnaissance, progressing to SMB-based deployment of a web shell, and creation of a reverse-shell connection. Memory analysis revealed a malicious executable that launched through the IIS process and made itself persistent through placing itself in the Windows Startup folder. 

The attack was severe enough to allow the attacker to gain the ability for remote code execution on the IIS server and established persistent malware execution, along with creating an outbound command-and-control channel. According to third party analysis, the recovered malware was capable of credential access activity across many browsers, although exfiltration and data theft was not confirmed through the provided evidence.

## Incident Scope

The system targeted for investigation was the public facing IIS server at IP `10.0.2.15`.
The suspicious activity and suspected attacker system was `10.0.2.4`.

Analysis was performed on the three provided forensic artifacts:
    - Network Packet Capture
    - Full memory image of the IIS server
    - `updatenow.exe` malware sample

The investigation consisted of discovering reconnaissance activity, HTTP and SMB activity, payload deployment, reverse-shell communication, process execution, persistence, and malware behavior.

No additional telemetry, logs, or artifacts were available for the simulation.

## Evidence Examined

The evidence artifacts provided and examined for this simulation were:
- **E002 `capture.pcapng` - Network Capture**
  A packet capture used to uncover network based reconnaissance activity, HTTP enumeration, SMB access, payload transfer, and reverse-shell activity

- **E003 `memdump.mem` - Memory Image**
 A memory image from the IIS server used to identify the process associated with the reverse-shell connection, examine process relationships, and determine where `updatenow.exe` was executing from.

- **E004 `updatenow.exe` - Malicious Executable**
 The SHA-256 hash was used to retrieve VirusTotal threat-intelligence and sandbox results identifying the malware family, packing method, network indicators, and observed capabilities.

 Original artifacts were preserved and hashed. Working copies were used for analysis.

## Attack Timeline
| Time | Event | Evidence |
|---|---|---|
| Earlier in capture | `10.0.2.4` initiated rapid TCP connection attempts against numerous ports on `10.0.2.15`, consistent with automated reconnaissance. | E002 — PCAP |
| Earlier in capture | HTTP enumeration from `10.0.2.4` identified the Nmap Scripting Engine through the HTTP User-Agent. | E002 — PCAP |
| Later | `10.0.2.4` accessed SMB shares `\\10.0.2.15\IPC$` and `\\10.0.2.15\Documents`. | E002 — PCAP |
| ~342.61s | An SMB2 CREATE and WRITE sequence transferred `shell.aspx` to the IIS server. | E002 — PCAP |
| ~389.51s | `10.0.2.4` issued an HTTP GET request for `/Documents/shell.aspx`, accessing the newly uploaded payload. | E002 — PCAP |
| ~401.50s | `10.0.2.15` initiated a TCP connection to `10.0.2.4:4443`. The resulting session contained sustained bidirectional traffic consistent with a reverse shell. | E002 — PCAP |
| Present in memory at acquisition | The connection `10.0.2.15:49688 → 10.0.2.4:4443` was associated with `w3wp.exe` PID 4332. | E003 — Memory Image |
| Present in memory at acquisition | `w3wp.exe` PID 4332 was observed
| Present in memory at acquisition | `w3wp.exe` PID 4332 spawned `updatenow.exe` PID 900. | E003 — Memory Image |
| Present in memory at acquisition | `updatenow.exe` executed from the Windows Startup folder, consistent with persistence. | E003 — Memory Image |

## Key Findings

### F-001 — Network Reconnaissance

**Evidence:** E002 - Network Capture

`10.0.2.4` performed rapid TCP connection attempts across numerous ports of `10.0.2.15`. Following this reconnaissance, there were HTTP requests containing a user agent identifying the Nmap Scripting Engine.

**Assessment:** The activity was consistent with automated network and HTTP reconnaissance.

**Supporting Exhibit:**
![synfloodrecon](/screenshots/SynFloodRecon.png)
![userAgentNmapScripting](/screenshots/UserAgentNmapScripting.png)

### F-002 — HTTP Enumeration

**Evidence:** E002 - Network Capture 

The attacker system accessed SMB shares `\\10.0.2.15\IPC$` and `\\10.0.2.15\Documents`.

**Assessment:** The attacker used their findings from the reconaissance phase to gain access to network shares on the IIS server.

**Supporting Exhibit:**
![smbSharesAccessed](/screenshots/smbSharesAccessed.png)

### F-003 — Web Shell Deployment

**Evidence:** E002 - Network Capture

The network activity showed a CREATE request via SMB for `shell.aspx`. Then a WRITE request containing about 1MB of data. The attacker then requested `/Documents/shell.aspx` through the IIS server.

**Assessment:** This activity confirmed that the attacker successfully transferred a payload to the IIS server and accessed it.

**Supporting Exhibit:**
![smbCreateRequests](/screenshots/smbCreateRequests.png)
![smbWriteRequests](/screenshots/smbWriteRequest.png)

### F-004 — Reverse-Shell Communication Established

**Evidence:** E002 — Network Capture; E003 — Memory Image

The IIS server initiated a TCP connection from `10.0.2.15:49688` to `10.0.2.4:4443` that involved bidirectional communication. Memory analysis associated the connection with `w3wp.exe` PID 4332.

**Assessment:** This suggested a high probability of a reverse shell connection through the IIS server's compromised worker process.

**Supporting Exhibit:**
![biderectionalCommunications](/screenshots/bidirectionalCommunications.png)
![volatilitynetscan](/screenshots/volatilityFilteredNetscanOutput.png)

### F-005 — Malicious Executable Launched Through IIS

**Evidence:** E003 — Memory Image

Process-tree analysis showed `w3wp.exe` PID 4332 had a child process: `updatenow.exe` PID 900.

**Assessment:** The parent-child relationship connects execution of
`updatenow.exe` directly to the compromised IIS application process and
supports post-exploitation code execution through the web server.

**Supporting Exhibit:**
![PSTree](/screenshots/volatilityWindowsPStree.png)

### F-006 — Startup Folder Persistence

**Evidence:** E003 — Memory Image

Command-line analysis showed `updatenow.exe` executing from:

`C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\updatenow.exe`

**Assessment:** Putting the executable inside the startup folder indicates an attempt to maintain persistence through future logins.

**Supporting Exhibit:**
![updateNowLaunchLocation](/screenshots/volatilityCommandLineUpdateNowLaunchLocation.png)

### F-007 — Recovered Executable Confirmed Malicious

**Evidence:** E004 — `updatenow.exe`; external threat intelligence

The SHA-256 hash of `updatenow.exe` was identified as malicious by the majority of VirusTotal scanners. Threat-intelligence and sandbox analysis associated the sample with Agent Tesla, UPX packing, credential-access behavior, process injection, and external network communication.

**Assessment:** External threat intelligence found the executable to be malicious. 

**Supporting Exhibit:**
![virusTotalResults](/screenshots/virusTotalResults.png)

## MITRE ATT&CK Mapping

These ATT&CK mappings are based on techniques supported by evidence in the provided artifacts. Other behaviors are excluded, as they have not been independently confirmed through the artifacts and would be supported only by third-party threat intelligence.

| Tactic | Technique | ID | Evidence |
|---|---|---|---|
| Reconnaissance | Active Scanning; Vulnerability Scanning | T1595.002 | Rapid TCP probing and Nmap NSE-based HTTP enumeration. |
| Command and Control | Ingress Tool Transfer | T1105 | `shell.aspx` was transferred by the attacker through SMB. |
| Persistence | Server Software Component; Web Shell | T1505.003 | `shell.aspx` was placed in a web-accessible directory and requested through IIS. |
| Command and Control | Non-Application Layer Protocol | T1095 | A bidirectional TCP session was established from the IIS server to `10.0.2.4:4443` following execution of the web shell. |
| Persistence / Privilege Escalation | Registry Run Keys / Startup Folder | T1547.001 | `updatenow.exe` executed from the system-wide Windows Startup directory. |
| Stealth | Software Packing | T1027.002 | VirusTotal identified `updatenow.exe` as UPX-packed. |


## Containment Recommendations
- Isolate the compromised IIS server, maintaining only necessary forensic examination access.
- Block all communication to the attacker's systems.
- Disable accounts if credential compromise is possible.
- Search other systems and the network telemetry for further potentially compromised systems.

## Eradication
- Remove `shell.aspx` and `updatenow.exe`.
- Remove any additional malicious files and persistence mechanisms uncovered through investigation.
- Patch and harden the services on the IIS that were exposed.
- Reset any accounts and change credentials that were potentially exposed.
- Search the rest of the enterprise for related attacker activity.

## Recovery
- Restore the server from a known good state.
- Validate malicious files are no longer present.
- Confirm there are no unauthorized outbound connections.
- Bring the systems back to production only after controls, patches, and monitoring are verified to be back in working order.
- Increase monitoring of the affected server post-recovery.

## Limitations
- Only three artifacts were provided, limiting the scope.
- No live access to the impacted server was possible.
- There were several other capabilities of the malware according to external threat-intelligence that could not be treated as confirmed or denied based on the provided forensic data.

## Lessons Learned
- The public facing IIS server should have been more hardened to reduce exposure, especially administrative protocols like SMB.
- Outbound connections from the IIS server should be monitored for suspicious activity. The connections made in this attack did not represent typical IIS activity and should have been actively monitored.
- The attacker was able to place the malicious ASPX file in a web-accessible location and attempt to create persistence in the Windows Startup folder. Locations like these should be monitored for for unexpected file creation, modification, and deletion.
- Finding correlation between the memory dump and packet capture increased confidence in the investigation. The packet capture identified the suspicious activity. The memory analysis was able to associate that connection with `w3wp.exe` and linked the IIS process to execution of `updatenow.exe`.
- External threat-intelligence was useful for identifying capabilities of the malware, but the provided findings need to be kept separated from the real incident-confirmed findings unless corroborated by the evidence.
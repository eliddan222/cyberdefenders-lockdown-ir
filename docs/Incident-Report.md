# Incident Response Report — INC-2026-001

## Executive Summary
TechNova System's SOC detected suspicious outbound traffic from the organizations public facing IIS server, that was suggestive of web-shell connections to an unknown host. The SOC analyst discovered three artifacts and presented them to the forensic examiner for investigation. These artifacts were a packet capture, a memory image, and a recovered executable. Analysis confirmed that the IIS server was compromised during a string of events beginning with automated reconnaisianse, progressing to SMB-based deployment of a web shell, and creation of a reverse-shell connection. Memory analysis revealed a malicious executable that launched through the IIS process and made itself persistent through placing itself in the Windows Startup folder. 

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
 Using its SHA256 hash static analysis via VirusTotal's external sandbox results identified the malware family, packing method, network indicators, and observed capabilities

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

## Key Findings

### F-001 — Network Reconnaissance
### F-002 — HTTP Enumeration
### F-003 — Unauthorized SMB Access
### F-004 — Web-Shell Deployment
### F-005 — Reverse-Shell Communication
### F-006 — IIS Process Correlation
### F-007 — Persistence via Startup Folder
### F-008 — Malicious Payload Attribution

## Indicators of Compromise

## MITRE ATT&CK Mapping

## Containment Recommendations

## Eradication and Recovery Recommendations

## Limitations

## Lessons Learned
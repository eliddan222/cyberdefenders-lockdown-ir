# CyberDefenders Lockdown - DFIR Investigation
A simulated digital forensics and incident response investigation of a compromised public-facing IIS server. 

This project used a documented incident-response SOP, packet capture analysis, memory image analysis, and a recovered malware sample.

## Investigation Summary

Packet capture analysis uncovered automated reconnaissance against the IIS server, followed by SMB deployment of an ASPX web shell, reverse-shell connection, execution of a malicious payload through the IIS worker process, and attempted persistence through the Windows Startup folder.

The provided packet capture and memory dump were able to be correlated to reconstruct the attack and determine which behaviors could be confirmed as opposed to only observed by third-party analysis.

![correlatedMemoryToNetwork](/screenshots/volatilityFilteredNetscanOutput.png)

## Tools
- Wireshark
- Volatility 3
- VirusTotal
- MITRE ATT&CK
- PowerShell

## Skills Applied
- Network forensics
- Windows memory forensics
- Evidence preservation
- Incident timeline reconstruction
- Cross-artifact correlation
- Malware triage
- MITRE ATT&CK Mapping
- IR documentation

## Documentation
- [Incident Response Report](docs/Incident-Report.md)
- [Incident Response SOP](docs/IR-SOP.md)
- [Analyst Case Notes](logs/analyst-notes.md)
- [Evidence Integrity Log](logs/evidence-log.csv)

## Works Cited
- CyberDefenders Lockdown: https://cyberdefenders.org/blueteam-ctf-challenges/lockdown/
- NIST SP 800-61 Rev. 3
- CISA Cybersecurity Incident & Vulnerability Response Playbooks
- MITRE ATT&CK



## Copyright
© 2026 Elijah D. All rights reserved.

This repository contains original analysis and documentation created for a CyberDefenders simulated lab. CyberDefenders and the Lockdown lab remain property of their respective owners.
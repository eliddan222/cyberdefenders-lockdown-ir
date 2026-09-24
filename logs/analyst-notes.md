# INC-2026-001 - Cyberdefender's Lockdown

Case Opened: 2026-09-20 16:56
Analyst: Elijah D.
Environment: CyberDefender's simulated incident

Initial Reason for Investigation:
The SOC detected suspicious outbound traffic from a public-facing IIS server. Initial activity was suggestive of possible web-shell deployment and covert communication with an unknown external host

Known Assets:
- Public Facing IIS server
- Additional affected systems/accounts unknown at initiation

Initial Severity:
Medium

Known Assets:
Unknown at initiation

Initial Objectives:
- Determine if a compromise occurred
- Identify affected systems and accounts
- Reconstruct attacker activity
- Determine the scope and impact of the incident
- Identify appropriate response actions

Limitations:
Analysis is limited to the provided artifacts.

## 2026-09-20 18:44
**Action:** Performed evidence intake for the supplied Lockdown archive.
**Result:** Recorded the SHA-256 hash of the original archive, extracted the
provided forensic artifacts, and inventoried them for analysis.

## 2026-09-20 18:53
**Action:** Created working copies of the known-good archive and extracted forensic artifacts for analysis.
**Result:** Compared SHA-256 hashes of the original evidence and working copies to verify the copies were identical before analysis.
**Assessment:** Original evidence was preserved and working copies were verified for analysis.
**Next Step:** Begin initial triage of the provided PCAP and document notable network activity.

## 2026-09-20 19:01
**Action:** Began initial triage of the supplied packet capture using Wireshark
**Objective:** Analyze the supplied network capture to identify the source and nature of reconnaissance activity, determine how the IIS service was enumerated, reconstruct how the attacker accessed the exposed network, identify any payload transfer to the server, and characterize any command-and control or reverse-shell communications. 

## 2026-09-20 19:05
**Observation:** Conversation statistics identified unusually high traffic volume between 10.0.2.4 and 10.0.2.15, totaling approximately 6,000 packets.
**Assessment:** This warrants additional review to determine if it is associated with the reported reconnaissance activity.
**Next Step:** Review TCP connection attempts and HTTP requests between the two hosts.

## 2026-09-20 19:15
**Observation:** Host 10.0.2.4 initiated rapid TCP connection attempts to numerous ports on 10.0.2.15 within a span of several seconds. Early traffic targeted TCP/80 and TCP/445
**Assessment:** The high number of connection attempts across many destination ports in such a short span of time is consistent with automated TCP reconnaissance against the IIS host.
**Next Step:** Examine HTTP traffic from the same source to determine the method used for web enumeration.
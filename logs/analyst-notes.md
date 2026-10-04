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
**Observation:** Host 10.0.2.4 initiated rapid TCP connection attempts to numerous ports on 10.0.2.15 within a span of several seconds. Early traffic targeted TCP/80 and TCP/445.
**Assessment:** The high number of connection attempts across many destination ports in such a short span of time is consistent with automated TCP reconnaissance against the IIS host.
**Next Step:** Examine HTTP traffic from the same source to determine the method used for web enumeration.

## 2026-10-03 16:09
**Observation**: Upon examining an HTTP packet sent from 10.0.2.4 to 10.0.2.15, it included a User-Agent that identified the client as the Nmap Scripting Engine.
**Assessment**: The HTTP enumeration was performed using Nmap scripting, which corroborates with the previous TCP reconnaissance observed from 10.0.2.4.
**Next Step**: Review the SMB traffic between the two hosts to identify which shares were accessed and whether a payload was transferred.

## 2026-10-03 16:17
**Observation:** SMB2 Tree connect requests from 10.0.2.4 to the IIS server found access to the `\\10.0.2.15\IPC$` and `\\10.0.2.15\Documents`.
**Assessment:** The reconnaissance source accessed SMB shares on the IIS server.
**Next Step:** Review any file operations that occurred over SMB to determine if a payload was transferred.

## 2026-10-03 16:20
**Observation:** Upon inspecting the SMB requests, there is create requests for files called `shell.aspx` and information.txt on the share accessed by 10.0.2.4.
**Assessment:** The filename and surrounding activity are consistent with attempted web-shell deployment, but file-write activity needs to be confirmed before confirming the completion of the transfer.
**Next Step:** Review the SMB2 write requests that occurred after the create request.

## 2026-10-03 16:24
**Observation:** 10.0.2.4 issued both an SMB2 CREATE request followed by a WRITE request to the accessed shares and wrote approximately 1MB of data.
**Assessment:** The CREATE and WRITE request sequence confirms that `shell.aspx` was transferred to the IIS server through SMB. The filename and HTTP request for `\Documents\shell.aspx` are consistent with the deployment of a web-accessible payload.
**Next Step:** Correlate the SMB upload with subsequent HTTP requests for `shell.aspx` and review traffic for reverse-shell activity.

## 2026-10-03 16:30
**Observation:** 47 seconds after `shell.aspx` was written to the IIS server over SMB, 10.0.2.4 initiated an HTTP GET request for /Documents/shell.aspx. 
**Assessment:** The timing and sequence indicate the attacker accessed the payload through the web server.
**Next Step:** Look at outbound TCP connections from 10.0.2.15 following this GET request to identify reverse-shell activity.

## 2026-10-03 16:45
**Observation:** About 12 seconds following the HTTP request for `shell.aspx`, the IIS server at 10.0.2.15 initiated a new TCP connection to 10.0.2.4 on port 4443.
**Assessment:** The timing and direction of the connection are consistent with the uploaded web shell initiating a reverse shell to the attacker controlled system on TCP/4443
**Next Step:** Inspect the resulting TCP stream to verify that the connection was established and determine if it contains command traffic.

## 2026-10-03 17:03
**Observation:** Following the HTTP request for `shell.aspx`, the IIS server established a sustained bidirectional communication TCP session from 10.0.2.15:49688 to 10.0.2.4:4443.
**Assessment:** The timing, connection direction, and continued data exchange are consistent with a reverse-shell. The application data is non-readable in the packet capture, so commands can't be verified from network evidence alone
**Next Step:** Correlate the TCP/4443 connection with the memory image to identify the process responsible for the connection.

## 2026-10-03 17:29
**Observation:** Using Volatility's `windows.netscan` a network connection from 10.0.2.15:49688 to 10.0.2.4:4443 was identified and associated with PID 4332 (`w3wp.exe`).
**Assessment:** This matches the previously identified reverse shell from the packet capture. Since `w3wp.exe` is the IIS worker process, the memory evidence strongly suggests that the uploaded payload executed through IIS and initiated the outbound connection to the attackers system.
**Next Step:** Examine PID 4332 and its process context to determine whether additional suspicious processes, activities, or tools were created.

## 2026-10-03 17:46
**Observation:** Using Volatility's `windows.pstree`, it was found that w3wp.exe (PID 4332) had a child process. The child process is updatenow.exe (PID 900). The executable path appears to reference the Windows Startup folder.
**Assessment:** The process relationship strongly suggests that the code executed through IIS caused updatenow.exe to run. Placement within the startup folder is indicative of attempted establishment of persistence.

## 2026-10-03 17:50
**Observation:** Volatility analysis identified `updatenow.exe` as a child of `w3wp.exe`. Command-line analysis confirmed that the executable was launched from `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\updatenow.exe`.
**Assessment:** Execution from the Windows Startup folder indicates the attacker was attempting to create persistence.
**Next Step:** Examine network activity associated with PID 900 to determine if `updatenow.exe` created additional connections.

## 2026-10-03 17:55
**Observation:** Volatility failed to find any network connections associated with PID 900. 
**Assessment:** Since the provided memory image doesn't provide evidence that `updatenow.exe` maintained a network connection at the time of acquision, its role will require further analysis of the recovered executable.
**Next Step:** Perform static analysis of the `updatenow.exe` sample file.

## 2026-10-03 18:01
**Observation:** A VirusTotal lookup of the SHA-256 hash of `updatenow.exe` returned a result of 57 detections from 70 vendors. The file was categorized as a trojan, with tags including debugger environment detection, long sleep behavior, WMI use, and UPX packing.
**Assessment:** This threat-intelligence shows that `updatenow.exe` is indeed malicious. This supports earlier findings in the memory evidence showing that the executable launched as a child process of the compromised IIS worker process. 
**Next Step:** Review available static and behavioral metadata for the malware sample to determine its capabilities and other indicators.

## 2026-10-03 18:09
**Observation:** VirusTotal sandbox results showed that `updatenow.exe` is associated with multiple MITRE ATT&CK techniques, including process injection, hidden-window execution, registry modification, credential access behavior, and use of non-standard network communications.
**Assessment:** These sandbox results indicate that the malware has stealth, persistence, and post-exploitation activity capabilities.
**Next Step:** Review VirusTotal behavioral details for specific proceses, registry, file, and network activity that can be correlated with the provided PCAP and memory evidence.

## 2026-10-03 18:20
**Observation:** VirusTotal behavioral analysis showed that `updatenow.exe` communicated with `cp8nl.hyperhost.ua` over TCP/587 and accessed credential storage associated with several web browsers. Threat-intelligence results identified the sample as Agent Tesla and static analysis indicated the binary was packed using UPX.
**Assessment:** The observed sandbox behavior is consistent with commodity credential theft software using SMTP-based communication. Agent Tesla attribution and the identified network activity are treated as external threat-intelligence findings unless independently corroborated using the supplied incident artifacts.
**Next Step:** Consolidate the confirmed incident findings, IOCs, and recommended containment and remediation actions into the final incident report.


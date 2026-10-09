# Standard Operating Procedure
### A basic SOP for incident response(IR) to be utilized for this lab.

## 1.0 Purpose & Scope
This document seeks to streamline and standardize the DFIR process during simulated incidents. This will allow for more uniform documentation of the response to the simulated incidents for easier reporting and analysis. It will also ensure the response process more closely aligns with real world production environment procedures, at least at a high level. This SOP was created specifically for simulated portfolio exercises and is not intended to represent an organization's production IR policy.

The scope of this document will be limited to simulated incidents involving virtualized endpoints and provided forensic artifacts. It covers endpoint, network, memory, and malware investigation, alongside incident triage, analysis, scoping, containment, eradication, recovery, and post-incident review.

Some actions performed during these simulations would typically require approval from management, system owners, legal teams, or other stakeholders in a production environment. For the purposes of the simulated environment, these actions will be treated as pre-approved and documented accordingly.

## 1.1 Roles & Responsibilities
I am acting as the incident analyst and forensic examiner. As mentioned previously in production environments I would need approval for several actions, but those actions will be treated as pre-approved.

The incident analyst will be responsible for verification that an incident has occurred, collecting and analyzing evidence, limiting damage, determining root causes when possible, restoring systems, and documenting actions taken during response. 

## 2.0 Case Initiation
1. Create a unique incident identifier.
2. Record the date and time the case was opened.
3. Record the reason for investigation (alerts, suspicious activity, reports, etc.).
4. Identify all known or suspected affected systems, accounts, or assets.
5. Record all sources of evidence available.
6. Assign an incident severity rating based on initial information.
7. Define initial objectives of the investigation.
8. Identify limitations in available evidence.
9. Create and maintain case notes and an incident timeline for documentation.

## 2.1 Evidence Intake and Preservation
1. Inventory all evidence received.
2. Assign each evidence source a unique identifier.
3. Record the original filename, file type, source, and other relevant information.
4. Hash the evidence with SHA-256 for applicable forensic data prior to analysis.
5. Preserve original evidence and use copies to avoid alteration of the original evidence where needed.
6. Document any limitations of acquisition or integrity concerns.

## 2.2 Initial Triage
1. Review the available evidence and initial incident information.
2. Determine whether the available information supports a suspected or confirmed security incident.
3. Identify known or suspected affected systems, accounts, and network assets.
4. Identify significant indicators or behaviors that require further investigation.
5. Reassess the incident severity based on information discovered during triage.
6. Identify evidence gaps or additional information needed to determine the incident scope.
7. Document initial observations and determine the next investigative actions.

## 2.3 Investigation and Analysis
1. Examine available network, endpoint, memory, and malware evidence relevant to the incident.
2. Correlate findings across multiple evidence sources when possible.
3. Record significant investigative actions, commands, filters, and observations in the analyst notes.
4. Identify attacker activity, affected assets, persistence mechanisms, command-and-control activity, and other relevant behaviors.
5. Maintain and update the incident timeline as significant events are identified.
6. Assign finding identifiers to confirmed or significant observations.
7. Map confirmed attacker behaviors to relevant MITRE ATT&CK techniques when applicable.
8. Document the confidence and limitations of significant findings.

## 2.4 Containment, Eradication, and Recovery
1. Identify actions needed to limit further attacker activity or prevent additional compromise.
2. Perform available containment actions within the simulated environment and document each action taken.
3. Identify malicious files, persistence mechanisms, compromised accounts, or other artifacts requiring removal or remediation.
4. Perform available eradication actions and document the results.
5. Identify any required credential resets, configuration changes, patches, or other remediation steps.
6. Restore affected systems or services when supported by the simulation.
7. Validate that identified malicious activity is no longer present.
8. Document any response actions that could not be performed and list them as recommended actions.

## 3.0 Case Closure and Lessons Learned
1. Confirm that all significant findings and response actions have been documented.
2. Verify that the incident timeline, evidence log, and case notes are complete.
3. Summarize the incident, affected assets, root cause, and overall impact.
4. Document any remaining risks, unresolved questions, or recommended follow-up actions.
5. Identify detection, response, or procedural improvements discovered during the investigation.
6. Update this SOP if lessons from the incident reveal a process that should be improved.
7. Mark the case as closed when investigation and documentation are complete.

References
- NIST SP 800-61 Rev. 3
- CISA Cybersecurity Incident & Vulnerability Response Playbooks
- MITRE ATT&CK

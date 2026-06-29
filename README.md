# Multi-Stage Insider Threat Investigation — ELK Stack Post-Incident Review

A full digital forensics and incident response (DFIR) investigation of a confirmed multi-stage network intrusion, conducted using the Elastic Stack (ELK). This project reconstructs a real-world style attack chain from raw Windows event log telemetry, maps every stage to the Lockheed Martin Cyber Kill Chain® and MITRE ATT&CK® frameworks, and concludes with evidence-based mitigation recommendations.

## Scenario

Acting as Lead Incident Responder for a simulated medium-sized enterprise, I investigated a confirmed breach of a Windows workstation flagged by the SOC for suspicious behaviour. The investigation window spanned 4–8 April 2025, with the primary attack window identified as 5 April 2025, 22:00–23:00 AWST. All analysis was performed in Kibana against Winlogbeat-indexed Windows Event Logs, using Kibana Query Language (KQL).

## Key Findings

- **Initial access** via FTP shell activity under the Administrator account, ~40 minutes before the main attack
- **Persistence setup** — manual installation of an SSH backdoor (`sc start sshd`, SSH key generation) ahead of the credential attack
- **Brute-force credential attack** over SSH — 10 failed logons followed by a successful authentication
- **Command-and-control beaconing** via a disguised executable generating DNS traffic blended with legitimate domains, alongside anomalous traffic on port 4444 (consistent with Metasploit's default reverse-shell port)
- **Dual-destination data exfiltration** via SCP — one transfer to an out-of-range external IP, a second directly to the attacker's machine
- **Defence evasion** using a Living-off-the-Land (LOLBin) technique — `cmd.exe` repeatedly spawned by `notepad.exe` to avoid signature-based detection
- **Resource hijacking** — deployment of a cryptocurrency miner under SYSTEM privileges
- **Long-term persistence** — execution of a Puppet configuration management agent, indicating intent to maintain automated, recurring access

Insider threat involvement was assessed at **medium confidence** — the attacking host was physically present on the same internal subnet as the victim, but the use of Kali Linux tooling is equally consistent with an external attacker with local network access.

## Methodology

The investigation followed a structured, pivot-based approach — moving from broad scope down to specific process command lines:

1. **Baseline scoping** — subnet-wide filtering surfaced the two dominant hosts and an anomalous port appearing in over 17% of records
2. **Victim account pivot** — isolating all activity tied to the targeted user account to confirm the attack window
3. **Process tree analysis** — exposing full parent/child process chains to reconstruct how persistence was installed and abused
4. **Exfiltration and C2 confirmation** — inspecting transfer commands and connection metadata to confirm both exfiltration destinations and the attacker's identity

Every step — queries run, reasoning, and findings — was logged in a full running sheet to make the investigation independently reproducible.

## Frameworks Applied

Every confirmed stage of the attack was mapped against:

- **Lockheed Martin Cyber Kill Chain®** — Reconnaissance through Actions on Objectives
- **MITRE ATT&CK®** — including T1078 (Valid Accounts), T1110.003 (Password Spraying), T1571 (Non-Standard Port C2), T1048 (Exfiltration Over Alternative Protocol), T1218 (System Binary Proxy Execution), T1496 (Resource Hijacking), and T1072 (Software Deployment Tools)

Where a phase had no supporting evidence, this was explicitly documented rather than left unexplained.

## Tools & Techniques

- **Elastic Stack (Elasticsearch, Kibana, Winlogbeat)** deployed locally via Docker
- **Kibana Query Language (KQL)** for log filtering, pivoting, and correlation
- Manual process-tree reconstruction from Windows Event IDs (4624, 4625, 4634, 1, 22, 7045)

## Limitations

- A log coverage gap meant the full extent of post-compromise impact could not be confirmed
- Single telemetry source (Winlogbeat only) — no packet-level corroboration available for C2 or exfiltration volume
- Some process command-line fields were truncated, limiting full command attribution for certain LOLBin activity

## Why This Project Matters

This was my first end-to-end SIEM-based incident investigation. It pushed me to think like an analyst — anchoring every claim to a timestamp and log ID, explicitly documenting where evidence was absent rather than guessing, and building a defensible chain of evidence from first compromise through to long-term persistence. It directly reflects the kind of detection-and-response work I want to do professionally.

## References

- [Lockheed Martin Cyber Kill Chain®](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)
- [MITRE ATT&CK®](https://attack.mitre.org/)
- [Kibana Query Language — Elastic Docs](https://www.elastic.co/docs/explore-analyze/query-filter/languages/kql)

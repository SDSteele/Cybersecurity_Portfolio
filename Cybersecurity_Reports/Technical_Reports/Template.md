# Penetration Test — Technical Report

**Client:** [Client Name]  
**Engagement:** [Engagement Title]  
**Date:** [Assessment Date]  
**Tester(s):** [Your Name / Team]  
**Type:** [External (EPT) / Internal (IPT)]

---

## Table of Contents

1. [Scope & Methodology](#1-scope--methodology)  
2. [Summary of Findings](#2-summary-of-findings)  
3. [Detailed Findings](#3-detailed-findings)  
4. [Attack Narrative (Chronological Walkthrough)](#4-attack-narrative-chronological-walkthrough)  
5. [Tools & Commands](#5-tools--commands)  
6. [Recommendations (Technical)](#6-recommendations-technical)  
7. [Appendices & Evidence Reference](#7-appendices--evidence-reference)  

---

## 1. Scope & Methodology

**Scope documents:** See the `scope/` folder for ROE, IP ranges, exclusions, and testing objectives.

**In-scope targets:**  
- IPs / Hostnames:  
  - `10.10.11.87`  
  - [Add other hosts here]

**Out-of-scope / Exclusions:**  
- [List exclusions from scope/ROE.txt]

**Engagement type & timing:**  
- Type: `EPT` / `IPT`  
- Testing window: `[Start date] — [End date]`  
- Test rules: see `scope/ROE.txt`

**Methodology:** (check all that apply)  
- [ ] Reconnaissance (passive & active)  
- [ ] Enumeration  
- [ ] Vulnerability scanning  
- [ ] Manual verification  
- [ ] Exploitation (controlled)  
- [ ] Post-exploitation (pivoting, data access)  
- [ ] Cleanup

**Primary tools / frameworks used:**  
- Nmap (scans in `scans/`)  
- masscan  
- Nessus / OpenVAS (exports in `scans/`)  
- Burp Suite (exports in `scans/`)  
- Metasploit (logs in `logs/`)  
- CrackMapExec, smbclient, rpcclient  
- Custom scripts (see `tools/`)  

---

## 2. Summary of Findings

| Severity     | Count |
|--------------|-------|
| **Critical** | [0]   |
| **High**     | [0]   |
| **Medium**   | [0]   |
| **Low**      | [0]   |
| **Info**     | [0]   |

**Overall risk rating:** `[Low / Medium / High]`  

**Top exploited issues (short):**  
- [Finding #ID] — [Short one-line summary of the finding]  
- [Finding #ID] — [Short one-line summary]

---

## 3. Detailed Findings

> **How to read each finding:**  
> - **Affected Systems** lists the hosts/services.  
> - **Evidence** points to files in the project tree (e.g., `evidence/`, `logs/`, `scans/`).  
> - **Proof** contains commands, payloads, or screenshots.  
> - **Recommendation** is the remediation direction for engineering.

---

### Finding #1 — [Vulnerability Title]
**ID:** `F-001`  
**Severity:** `Critical` / `High` / `Medium` / `Low` / `Info`  
**Affected Systems:** `10.10.11.87:80 (http)` — `webserver.example.local`  
**CVE / Reference:** `[CVE-YYYY-NNNN]` or `CWE-XXXX` / OWASP reference

**Description**  
[Detailed technical description of the issue: how it was identified, vulnerable component, version, why it is vulnerable. Include protocol/port/service details.]

**Evidence**  
- Scan: `scans/nmap-10.10.11.87.xml`  
- Proof-of-concept payload or request: `logs/poc_f001.txt`  
- Screenshot: `evidence/screenshots/f001_poc.png`  
- Exfiltrated data (if any): `evidence/data/f001_exfil/`

**Proof of Exploitation**  
```bash
# Example commands used to confirm / exploit
nmap -sV -p 80 10.10.11.87 -oN scans/nmap-10.10.11.87.txt
curl -i -s -X POST 'http://10.10.11.87/vuln' -d 'payload=...'
# output saved to logs/poc_f001.txt
```

**Impact**  
[Technical impact: remote code execution, authentication bypass, data exfiltration, lateral movement capability, etc.]

**Likelihood of Exploitation**  
[Low / Medium / High — justify briefly]

**Remediation / Mitigation**  
1. Apply vendor patch: `Upgrade to package X.Y.Z or later`  
2. Configuration change: `Disable feature X`  
3. Compensating controls: `WAF rule, IPS signature`  
4. Verify: `After patch, confirm by running scans/nmap-... and retesting POC.`

**References**  
- [Vendor advisory link or CVE summary]  
- [Exploit-db or proof resource if applicable]

---

### Finding #2 — [Vulnerability Title]
*(Repeat structure above for each finding)*

---

## 4. Attack Narrative (Chronological Walkthrough)

A concise, step-by-step log of the engagement narrative, including where evidence is stored.

1. **Recon & Discovery**  
   - Date/time: `[timestamp]`  
   - Tools: Nmap (`scans/`), masscan  
   - Findings: Open ports — see `scans/nmap-initial.txt`

2. **Initial Access**  
   - Method: [e.g., SQL injection, default creds, exposed RDP]  
   - Evidence: `logs/initial_access.txt`, `evidence/screenshots/initial_access.png`

3. **Privilege Escalation**  
   - Method: [local exploit, sudo misconfig, kernel exploit]  
   - Evidence: `logs/privesc.txt`, `evidence/credentials/root_dump.txt`

4. **Lateral Movement**  
   - Hosts pivoted to: `[IP list]`  
   - Tools used: `wmiexec.py`, `smbclient`  
   - Evidence: `logs/lateral_movement.txt`

5. **Data Access / Exfiltration**  
   - Data accessed: `evidence/data/` (list files & locations)  
   - Evidence: `evidence/data/customer_emails.csv`

6. **Persistence & Cleanup**  
   - Persistence: `[if applied during test — otherwise note not attempted or reverted]`  
   - Cleanup steps performed: removed test artifacts, closed sessions (see `logs/cleanup.txt`)

---

## 5. Tools & Commands

**Full tool list and versions** (store installer copies or configs in `tools/`):
- Nmap `vX.Y` — scans in `scans/`
- masscan `vX.Y`
- Burp Suite `vX.Y` — project exports in `scans/burp/`
- Metasploit `vX.Y` — resource files in `tools/`
- Custom scripts — `tools/` (list names & purpose)

**Key commands / excerpts** (place full logs in `logs/`):
```bash
# Sample scan
nmap -sC -sV -oA scans/nmap-initial 10.10.11.87

# Brute force example (logged)
hydra -L users.txt -P passwords.txt ssh://10.10.11.87 -t 4 -o logs/hydra_ssh.txt

# Exploit sample (recorded)
python3 tools/exploit_xyz.py --target 10.10.11.87 --cmd 'whoami' > logs/exploit_xyz_out.txt
```

---

## 6. Recommendations (Technical)

Order these by priority: Immediate (0–7 days), Short (7–30 days), Medium (30–90 days), Long-term (>90 days).

**Immediate**  
- Patch vulnerable services (see Findings F-###).  
- Rotate credentials exposed during test and enforce strong passwords.  
- Apply temporary network restrictions to critical systems.

**Short Term**  
- Enforce MFA for all remote access.  
- Harden exposed services with secure configurations (disable old SSL/TLS, remove weak ciphers).  
- Implement host-based and network-based detection rules for the techniques observed.

**Medium Term**  
- Network segmentation: isolate management interfaces and sensitive data stores.  
- Deploy EDR and configure alerting for lateral movement patterns.  
- Run code review / secure development lifecycle for affected applications.

**Long Term**  
- Formalize vulnerability management (monthly scans + scheduled pentests).  
- Conduct employee training around phishing and credential hygiene.  
- Integrate pentest results into risk register and remediation tracking.

---

## 7. Appendices & Evidence Reference

**Project tree reference (example):**
```
Projects/
└── Acme Company/
    ├── EPT/ or IPT/
    │   ├── evidence/
    │   │   ├── credentials/
    │   │   ├── data/
    │   │   └── screenshots/
    │   ├── logs/
    │   ├── scans/
    │   ├── scope/
    │   └── tools/
```

**Appendix A — Scope Documents**  
- `scope/ROE.txt` — Rules of Engagement  
- `scope/scope_targets.txt` — Targets and IP ranges

**Appendix B — Scans**  
- `scans/nmap-initial.txt`  
- `scans/nessus-report.xml`

**Appendix C — Logs**  
- `logs/initial_access.txt`  
- `logs/privesc.txt`  
- `logs/cleanup.txt`

**Appendix D — Evidence**  
- Credentials: `evidence/credentials/` (treat as sensitive)  
- Data and exfiltrated files: `evidence/data/`  
- Screenshots: `evidence/screenshots/`

---

## Contact & Notes

For remediation assistance, retesting, or clarifications contact:  
**[Tester Name]** — [email@example.com] — [phone]

> **Security & handling note:** Evidence and credentials are sensitive. Keep `evidence/` off public repositories and store backups on encrypted media. Consider GPG encryption or an encrypted volume for the `evidence/` directory.

---

*End of technical report template — populate each section and replace placeholders with the specific details from the engagement. Store the completed report within the project root (e.g., `Projects/<Client>/<Engagement>/REPORTS/pentest-technical-report.md`) and link / zip the `evidence/` folder separately following your organization’s handling policy.*

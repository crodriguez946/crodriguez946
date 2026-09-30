# Carina Esparza | Cybersecurity Portfolio

📍 San Antonio, TX | 🔗 [LinkedIn](https://www.linkedin.com/in/carina-esparza-199994316) | ✉️ [summit.tiger1767@eagereverest.com](mailto:summit.tiger1767@eagereverest.com)

---

## 🛡️ About Me

Cybersecurity student with a strong operational background in compliance, risk management, and precise documentation, transitioning into hands-on technical defense. Bilingual, detail-oriented, and focused on **SOC Analysis, Threat Detection, and Governance, Risk & Compliance (GRC)**.

- 🎯 **Target Roles:** SOC Analyst (Tier 1) | GRC Analyst | Information Security Associate
- 🛠️ **Active Practice:** Home Labs, CTF Competitions, Blue Team & Threat Detection
- 🎓 **Education:** A.A.S. Cybersecurity | Member: Phi Theta Kappa | CIMA-LSAMP Scholar

---

## 🧰 Technical Skills & Tools

| Category | Tools & Competencies |
| :--- | :--- |
| **Defensive & Detection** | Wireshark, Splunk, Suricata, Sysmon, Log Triage, Incident Analysis |
| **Systems & Networking** | Linux (Ubuntu, CLI, Bash), Windows Server, TCP/IP, OSI Model, DNS, Subnetting |
| **Frameworks & Governance** | NIST CSF, MITRE ATT&CK, HIPAA Compliance, Access Controls (RBAC), Zero Trust |
| **Hands-On Practice** | TryHackMe, Hack The Box, LetsDefend, VirtualBox |

---

## 🚀 Featured Projects & Lab Work

### 1. Digital Forensics & Artifact Extraction (Holmes CTF 2026)
> **Objective:** Conduct forensic triage on a 13.9 GB `.E01` disk image to trace VPN configurations, parse raw database artifacts, and extract compromised credentials.
- **Environment:** FTK Imager, PowerShell CLI, Regular Expressions (`regex`), Windows File System (`NTFS`).
- **Key Actions:**
  - Mounted and navigated a 13.9 GB `.E01` forensic image; isolated user activity profiles and staging notes (`todo.txt`).
  - Extracted OpenVPN telemetry (`spur.log`) to uncover external server IPs, assigned tunnel addresses, and certificate CNs.
  - Scripted a custom PowerShell string-carving routine to parse unindexed SQLite application databases (`Logs.db`).
  - Recovered cleartext credentials (`spurio9@murknet.htb`) through forensic text-pattern reconstruction.
- 📄 **[Read Full Forensic Investigation Report](./reports/ctf-reichenbach-forensics.md)**

---

### 2. Network Traffic Analysis & Packet Inspection
> **Objective:** Investigate suspicious network activity using Wireshark and command-line packet tools to detect anomalies and protocol abuse.
- **Environment:** Ubuntu Linux, Wireshark, `tshark`.
- **Key Actions:**
  - Analyzed packet captures (.pcap) to trace connection sequences, handshakes, and payload deliveries.
  - Filtered DNS and HTTP traffic to detect anomalous outbound requests and potential malicious beaconing.
  - Extracted transmission artifacts and documented Indicators of Compromise (IoCs) in a structured incident summary.
- **Key Finding:** Identified unauthorized external requests by isolating protocol timing irregularities.
- 📄 **[Read Full Incident Report Write-Up](./reports/incident-report-dns-triage.md)**

---

### 3. Linux System Administration & Security Hardening
> **Objective:** Deploy, configure, and harden a customized Linux virtual workstation for daily security analysis and tooling.
- **Environment:** Ubuntu Linux, Kitty Terminal, Bash.
- **Key Actions:**
  - Configured user access controls, file permissions (`chmod`, `chown`), and secure SSH configurations.
  - Automated routine administrative tasks and system status monitoring using custom Bash shell scripting.
  - Implemented environment workflows and security audit checks on running processes and network sockets (`netstat`, `ss`).
- **Key Finding:** Reduced system attack surface by disabling unused background daemons and enforcing least-privilege permissions.

---

### 4. Healthcare Compliance & Access Risk Evaluation (GRC)
> **Objective:** Evaluate security controls and access management policies for sensitive patient and transaction data.
- **Key Actions:**
  - Audited role-based access control (RBAC) separation to mitigate privilege creep across sensitive records.
  - Assessed procedural workflows against HIPAA privacy/security rules and NIST CSF core functions.
  - Formulated remediation recommendations addressing data confidentiality, integrity, and audit logging.

---

## 🏆 Certifications & Affiliations

- **Pre-Security Certification** — TryHackMe
- **Affiliations:** NightHax Cybersecurity Club | DEF CON SATX Group | Phi Theta Kappa Honor Society

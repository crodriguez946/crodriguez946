# Forensic Investigation Report: Disk Image & Artifact Analysis
**Operation / Event:** Holmes CTF 2026 — The Reichenbach Directive (Sep 2026)  
**Analyst:** Carina Esparza  
**Artifact Analyzed:** 13.9 GB Windows Disk Image (`.E01` Forensic Format)  
**Tooling:** FTK Imager, PowerShell, SQLite, Regular Expressions (`regex`)

---

## 1. Executive Summary
Conducted a post-incident forensic disk analysis of an acquired 13.9 GB `.E01` forensic image. The objective was to reconstruct user activity, trace network communications, extract configuration databases, and isolate compromised credential sets. Forensic triage identified an established OpenVPN tunnel endpoint, extracted persistent application databases, and recovered cleartext credentials from unindexed SQLite log stores using PowerShell text extraction.

---

## 2. Evidence Acquisition & File System Reconnaissance
- **Evidence Container:** 13.9 GB Expert Witness Format image (`.E01`) mounted via FTK Imager.
- **Target Profiles:** Inspected artifacts under user profiles `spur` and `Administrator`.
- **System Version:** Identified `VERSION.txt` (`v1.0.0-alpha`).
- **Operational Guidance:** Recovered `todo.txt` from desktop, detailing user staging:
  1. *Connect to OpenVPN*
  2. *Enter the chat*
  3. *Wait to be invited*

---

## 3. Network Configuration & VPN Telemetry
Forensic parsing of application directories identified an installer payload (`OpenVPN-2.7.6-I001-amd64.msi`) alongside active connection remnants in `spur.log`:

| Parameter | Observed Value | Forensic Significance |
| :--- | :--- | :--- |
| **VPN Endpoint** | `18.156.81.166:7577` | External infrastructure host / listener |
| **Tunnel Assigned IP** | `10.129.175.2` | Internal overlay address assigned to victim |
| **Gateway IP** | `10.129.175.1` | Local transit node |
| **Cert Common Names** | `NPLN-CA`, `NPLN-VPN-7577` | Cryptographic trust anchors for infrastructure |

---

## 4. Database Extraction & Automated Credential Recovery
Inspected user application directories (`UserData` and `Data`), isolating multiple SQLite databases:
- `Logs.db`
- `omemo_spurio9@murknet.htb.db`
- `openpgp.db`
- `Settings.sqlite`

### Automated String Scraping via PowerShell
Because standard tabular querying was restricted, executed a low-level byte read and ASCII printable-string extraction using custom regex pattern matching:

```powershell
# Extract printable ASCII sequences (>= 4 chars) from raw SQLite binary
$bytes = [System.IO.File]::ReadAllBytes("C:\Users\Administrator\Desktop\UserData\Logs.db")
$text  = [System.Text.Encoding]::ASCII.GetString($bytes)
[regex]::Matches($text, "[\x20-\x7E]{4,}") | ForEach-Object { $_.Value } | Out-File "C:\Users\Administrator\Desktop\chat_raw.txt"

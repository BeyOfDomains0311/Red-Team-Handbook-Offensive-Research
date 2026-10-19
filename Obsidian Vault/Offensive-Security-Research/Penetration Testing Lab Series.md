# README — Penetration Testing Lab Series
## CompTIA PenTest+ (PT0-003) — Authorized Lab Environment

---

## Overview

This lab series covers the full penetration testing engagement lifecycle from passive reconnaissance through domain compromise and professional reporting. Each lab builds directly on the previous one — moving from passive intelligence gathering through active enumeration, vulnerability analysis, exploitation, post-exploitation, lateral movement, and reporting.

The objective is NOT to collect flags or complete checkboxes.

The objective is to develop a **repeatable, transferable methodology** that can be applied to any unfamiliar authorized environment.

---

## Lab Series Index

| Lab | Title | PTES Phase | Core Tools |
|-----|-------|-----------|------------|
| Lab 01 | Penetration Testing Information Gathering Techniques | Phase 2 — Reconnaissance | DNSDumpster, Whois, Netdiscover, Nmap, Netcat |
| Lab 02 | Enumeration Tools and Techniques | Phase 3 — Enumeration | Nmap NSE, OWASP ZAP, smbclient |
| Lab 03 | Vulnerability Scanning Techniques | Phase 4 — Vulnerability Analysis | OpenVAS/GVM, Nikto, ipcalc |
| Lab 04 | Exploitation Techniques | Phase 5 — Exploitation | SQLMap, Burp Suite, Metasploit, Hydra |
| Lab 05 | Post-Exploitation and Privilege Escalation | Phase 6 — Post-Exploitation | Meterpreter, Impacket secretsdump, WinPEAS, LinPEAS |
| Lab 06 | Lateral Movement, Active Directory Attacks, and Reporting | Phases 7–10 — Lateral Movement / Objective / Evidence / Report | NetExec, BloodHound, Impacket, Evil-WinRM, Hashcat |

---

## Prerequisites

### Technical Prerequisites

- Linux command-line proficiency (navigation, file operations, pipes, redirection)
- TCP/IP model understanding (minimum Layers 2–4)
- Basic networking: IP addressing, subnetting, ports, protocols
- Kali Linux VM operational in the lab environment

### Lab Environment Prerequisites

- **ACIKALI VM** — Attack platform (Kali Linux) — 192.168.0.100
- **Server VM** — Target running XAMPP (Apache + MySQL + PHP) — 192.168.0.2
- **DC VM** — Domain Controller (Windows Server 2022) — 192.168.0.3
- Network connectivity confirmed between attacker and all targets

---

## Lab Environment Topology

```
[ ACIKALI VM ] — Attacker (192.168.0.100)
        │
        └─── 192.168.0.0/24 (Lab Subnet)
                    │
                    ├── 192.168.0.1  — Gateway / Router
                    ├── 192.168.0.2  — Server VM (XAMPP — Apache, MySQL, PHP, DVWA)
                    │                  Hostname: ACISRV01
                    └── 192.168.0.3  — DC VM (Windows Server 2022 — Domain Controller)
                                       Hostname: ACIDC01.ACIPLAB.com
```

> **Note:** IP addresses vary by lab instance. Always confirm active hosts with `sudo nmap -sS 192.168.0.0/24` before beginning any enumeration activity.

---

## Tools Used Across This Series

| Tool | Purpose | First Used |
|------|---------|-----------|
| DNSDumpster | Passive DNS reconnaissance | Lab 01 |
| Whois | Domain and IP registration lookup | Lab 01 |
| Netdiscover | Passive and active ARP-based host discovery | Lab 01 |
| Nmap | Port scanning, OS detection, version enumeration, NSE scripting | Lab 01 |
| Netcat (nc) | Raw TCP connections, banner grabbing, reverse shell listener | Lab 01 |
| OWASP ZAP | Automated web application vulnerability scanning | Lab 02 |
| smbclient | SMB share listing and unauthenticated access testing | Lab 02 |
| ipcalc | Subnet scope calculation and confirmation | Lab 03 |
| OpenVAS / GVM | Comprehensive host-level CVE vulnerability scanning | Lab 03 |
| Nikto | Web server and web application vulnerability scanning | Lab 03 |
| Burp Suite | Intercepting proxy, manual HTTP request manipulation | Lab 04 |
| SQLMap | Automated SQL injection detection and exploitation | Lab 04 |
| Hydra | Online credential brute-forcing | Lab 04 |
| Metasploit / msfconsole | Exploitation framework, session management | Lab 04 |
| Meterpreter | In-memory post-exploitation agent | Lab 05 |
| Impacket secretsdump | Remote SAM and NTDS credential extraction | Lab 05 |
| WinPEAS | Automated Windows privilege escalation enumeration | Lab 05 |
| LinPEAS | Automated Linux privilege escalation enumeration | Lab 05 |
| NetExec (nxc) | Authenticated lateral movement testing over SMB/WinRM/LDAP | Lab 06 |
| Evil-WinRM | WinRM-based interactive shell access | Lab 06 |
| BloodHound + SharpHound | Active Directory attack path collection and analysis | Lab 06 |
| Impacket GetUserSPNs | Kerberoasting — SPN enumeration and TGS ticket request | Lab 06 |
| Hashcat | Offline hash cracking | Lab 06 |

---

## PTES Methodology Reference

```
PHASE 1 — PRE-ENGAGEMENT
    Scope definition, objectives, rules of engagement, authorization confirmation

PHASE 2 — RECONNAISSANCE
    Passive:  DNSDumpster → Whois → Netdiscover -p
    Active:   Nmap -sS → Nmap -O → Nmap -sV → Nmap --script → Netcat

PHASE 3 — ENUMERATION
    Web:      OWASP ZAP → Nmap --script http-enum → Nmap --script http-title
    SMB:      Nmap --script smb-os-discovery → Nmap --script smb-enum-shares
              → smbclient -N -L → smbclient -N (share access attempt)

PHASE 4 — VULNERABILITY ANALYSIS
    Host:     greenbone-feed-sync → gvm-start → OpenVAS scan → CVE cross-reference
    Web:      nikto -h → nikto -Tuning 0 → nikto -Tuning 1
    Manual:   NVD → Exploit-DB → MITRE ATT&CK

PHASE 5 — EXPLOITATION
    Web:      Burp Suite Repeater (manual SQLi) → SQLMap (automated)
    OS:       DVWA Command Injection → command separator injection → reverse shell
    Upload:   File upload → PHP web shell → Netcat reverse shell upgrade
    Auth:     Hydra HTTP POST brute-force → credential validation
    Network:  msfconsole → exploit module → payload selection → session establishment

PHASE 6 — POST-EXPLOITATION
    Orient:   sysinfo → getuid → getpid → ps → migrate
    Enum:     whoami /priv → net user → netstat → ifconfig → route
    PrivEsc:  local_exploit_suggester → SeImpersonatePrivilege → PrintSpoofer
              → unquoted service paths → weak service permissions
    Creds:    hashdump → load kiwi → creds_all → lsa_dump_secrets
              → impacket-secretsdump (remote)
    Linux:    sudo -l → find SUID → getcap → cron analysis

PHASE 7 — LATERAL MOVEMENT
    Auth:     NetExec SMB spray → Pass-the-Hash (psexec / wmiexec)
    Shell:    Evil-WinRM → interactive domain session
    AD:       bloodhound-python → SharpHound → BloodHound GUI analysis
    Attack:   GetUserSPNs → TGS ticket extraction → Hashcat TGS crack

PHASE 8 — OBJECTIVE
    Domain:   impacket-secretsdump against DC → full NTDS dump
    Access:   Domain Administrator credential validation

PHASE 9 — EVIDENCE
    Collect:  Screenshots → command output → request/response pairs
    Format:   Title → Severity → Asset → Description → Reproduction
              → Evidence → Impact → Remediation → References

PHASE 10 — CLEANUP
    Remove:   Uploaded shells → staged binaries → created accounts
    Restore:  Modified files → disabled services → registry changes
    Terminate: Active sessions → listeners → background jobs
```

---

## Methodological Sequence — Per Lab

```
OBSERVATION
    ↓
HYPOTHESIS
    ↓
TEST
    ↓
RESULT
    ↓
UPDATED HYPOTHESIS
    ↓
EXPLOITATION (where applicable)
    ↓
VALIDATION
    ↓
IMPACT
```

Every scan, every flag, every tool invocation should serve this reasoning chain. Random enumeration builds no transferable skill.

---

## How to Use These Notes

1. Read **ELI5** — Get the concept without jargon
2. Read **For the Tech Pro** — Understand the technical depth
3. Study **Command Breakdown** — Know what every component does before executing
4. Check **Explore the Flag** — Understand what options are available
5. Review **Expected Output** — Know what success looks like before running the command
6. Execute the command — Compare your actual output to expected output
7. Read **Operator Notes** — Understand tactical significance
8. Work through **Mind Gap Analysis** — Confirm you understand, not just executed
9. Review **Key Takeaways** — Consolidate learning before moving to the next lab

---

## Environment Setup Quick Reference

### Before Every Lab

```bash
# Confirm network connectivity to target
ping -c 3 <TARGET_IP>

# Confirm active hosts in scope
sudo nmap -sS 192.168.0.0/24

# Document active host IPs before proceeding
```

### DVWA Setup (Labs 02+)

```
1. Server VM → XAMPP Control Panel
2. Confirm Apache and MySQL status: GREEN (Running)
3. Browser → http://192.168.0.2/dvwa
4. Login: admin / password
5. DVWA Security → Low → Submit
```

### GVM Setup (Lab 03)

```bash
sudo greenbone-feed-sync     # Sync CVE/NVT database — required before every scan
sudo gvm-start               # Start full GVM stack
# Access web UI: https://127.0.0.1:9392
# Accept self-signed TLS certificate warning
sudo gvm-stop                # Stop when complete
```

### BloodHound Setup (Lab 06)

```bash
sudo neo4j start             # Start Neo4j graph database
bloodhound &                 # Launch BloodHound GUI
# Default neo4j credentials: neo4j / neo4j (change on first login)
```

---

## Complete Kill Chain — This Lab Series

```
Lab 01 → Passive recon → Apache 2.4.58, DC ACIDC01.ACIPLAB.com identified
    ↓
Lab 02 → Enumeration → DVWA confirmed, SMB shares listed, DC confirmed via NETLOGON/SYSVOL
    ↓
Lab 03 → Vulnerability scan → CVEs correlated, file upload endpoint discovered
    ↓
Lab 04 → Exploitation → SQL injection → DB credential dump, web shell → SYSTEM shell
    ↓
Lab 05 → Post-exploitation → NTLM hashes extracted, service account credential in LSA secrets
    ↓
Lab 06 → Lateral movement → PTH to DC → Kerberoasting → NTDS dump → Domain Admin
              → Report → Cleanup
```

---

## Common Errors Reference

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| `Permission denied` on nmap | Missing sudo for raw socket operations | Prefix with `sudo` |
| `Unable to connect with SMB1` in smbclient | Server correctly rejecting legacy SMB1 | Expected — ignore, look at results above it |
| ZAP TLS certificate warning | GSAD self-signed certificate | Advanced → Accept the Risk and Continue |
| Nmap `-p 139, 445` misparse | Space after comma in port list | Use `-p 139,445` — no space |
| Netdiscover returns nothing | Passive mode — no ARP traffic in window | Switch to `nmap -sn 192.168.0.0/24` for active discovery |
| OpenVAS returns no findings | Stale NVT/CVE feed | Run `sudo greenbone-feed-sync` before every scan |
| GVM web UI unreachable | Stack not started | Run `sudo gvm-start` and wait for all services |
| SQLMap returns no injection | Stale or expired session cookie | Refresh DVWA session and copy new PHPSESSID |
| Meterpreter session dies | Unstable process hosting the session | Migrate to explorer.exe or svchost.exe before enumerating |
| PTH fails with Impacket | LM hash field missing | Format is `LM:NTLM` — use `aad3b435b51404eeaad3b435b51404ee:NTLM` or `:NTLM` |
| BloodHound shows empty graph | Data not imported | Drag and drop the SharpHound .zip file into the BloodHound GUI |
| Kerberoasting returns no SPNs | User lacks domain authentication | Confirm valid domain credentials and DC IP |

---

## Glossary

| Term | Definition |
|------|-----------|
| **ARP** | Address Resolution Protocol — maps IP addresses to MAC addresses at Layer 2 |
| **Banner Grabbing** | Reading service response headers to identify software name and version |
| **CVE** | Common Vulnerabilities and Exposures — standardized vulnerability identifier |
| **CVSS** | Common Vulnerability Scoring System — severity rating 0.0–10.0 |
| **DCSync** | Attack technique replicating the DC's credential database using legitimate AD replication protocols |
| **DPAPI** | Data Protection API — Windows mechanism for storing encrypted secrets |
| **FQDN** | Fully Qualified Domain Name — complete hostname including domain suffix |
| **GTFOBins** | Curated list of Unix binaries that can be abused for privilege escalation |
| **Kerberoasting** | Requesting TGS tickets for SPN-registered accounts and cracking them offline |
| **LSASS** | Local Security Authority Subsystem Service — Windows process storing credentials in memory |
| **NTLM** | NT LAN Manager — Windows challenge-response authentication protocol |
| **NTDS.dit** | Active Directory database file containing all domain account hashes |
| **NSE** | Nmap Scripting Engine — Lua-based plugin system extending Nmap |
| **NVT** | Network Vulnerability Test — an individual OpenVAS scan script |
| **OUI** | Organizationally Unique Identifier — first 3 bytes of a MAC address identifying the vendor |
| **Pass-the-Hash** | Authenticating using an NTLM hash directly without knowing the plaintext password |
| **PTES** | Penetration Testing Execution Standard — structured engagement methodology |
| **RID** | Relative Identifier — numeric user identifier within a Windows domain or local SAM |
| **SAM** | Security Account Manager — Windows database storing local account credential hashes |
| **SMB** | Server Message Block — Windows file and printer sharing protocol |
| **SPN** | Service Principal Name — unique identifier associating a service with a domain account |
| **SYSVOL** | System Volume — DC share hosting Group Policy Objects |
| **NETLOGON** | Domain Controller share hosting logon scripts |
| **TGS** | Ticket Granting Service — Kerberos ticket issued for access to a specific service |
| **TGT** | Ticket Granting Ticket — initial Kerberos ticket used to request service tickets |
| **UNC** | Universal Naming Convention — `\\server\share` (Windows) / `//server/share` (Linux) |
| **Null Session** | Anonymous SMB connection — no username or password provided |
| **WinRM** | Windows Remote Management — HTTP/HTTPS-based remote shell protocol (port 5985/5986) |

---

## Core Principles

> "Every open port is a hypothesis — not a confirmation."

> "Scanner output is an indication — not a finding. The chain is always: Indication → Validation → Exploitation → Impact."

> "Passive reconnaissance first. Always. Before sending a single packet to the target."

> "One source is an indication. Two corroborating sources begin to build evidence."

> "Version information is a CVE query waiting to happen."

> "Access denied is still intelligence — it confirms the resource exists."

> "An NTLM hash does not need to be cracked to be useful."

> "Understand what the tool is doing underneath. Never operate a tool you cannot explain."

> "SYSTEM on one host is a starting position — not the objective."

> "Every cracked or extracted credential is a lateral movement hypothesis. Test it everywhere."

---

*Lab series maintained for CompTIA PenTest+ (PT0-003) authorized laboratory instruction.*
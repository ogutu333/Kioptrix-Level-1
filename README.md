# Kioptrix Level 1 – Security Assessment

[![Pentest](https://img.shields.io/badge/Penetration%20Test-Kioptrix%20Level%201-blue)](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/)
[![Date](https://img.shields.io/badge/Date-August%202026-lightgrey)](https://github.com/)
[![Status](https://img.shields.io/badge/Status-Complete-brightgreen)]()

---

## Overview

This repository contains my penetration testing work on the [Kioptrix Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/) vulnerable VM from VulnHub. The main goal of the lab was to see if I could gain **root access** and document the process from start to finish.

I worked through the assessment step by step, starting with reconnaissance and enumeration, then moving on to vulnerability identification, exploitation, and post-exploitation. I also documented the commands I used, the results I got, and the security issues I found along the way.

---

## Key Findings

| Vulnerability                                        | Service                                      | Impact                                |
| ---------------------------------------------------- | -------------------------------------------- | ------------------------------------- |
| **Samba trans2open buffer overflow (CVE-2003-0201)** | Samba 2.2.x on port 139                      | **Critical** – immediate root access  |
| **mod_ssl / OpenSSL vulnerabilities**                | Apache 1.3.20, mod_ssl 2.8.4, OpenSSL 0.9.6b | **High** – remote code execution      |
| **SSHv1 protocol support**                           | OpenSSH 2.9p2                                | **Medium** – cryptographic weaknesses |
| **Outdated Linux kernel**                            | Linux 2.4.x                                  | **Medium** – multiple local exploits  |

**Overall Risk Rating:** **CRITICAL** – I was able to fully compromise the system within minutes.

---

## Methodology

I worked through the assessment in five main phases:

1. **Reconnaissance** – I started by discovering devices on the network using `netdiscover`, `arp-scan`, and `nmap` ping sweeps.
2. **Enumeration** – I performed a detailed port scan to identify running services, their versions, and the operating system using `nmap -sV -sC -O -A`.
3. **Vulnerability Assessment** – I looked at the services and versions I found and checked which ones could potentially be exploited.
4. **Exploitation** – I used Metasploit and the `exploit/linux/samba/trans2open` module to exploit the vulnerable Samba service.
5. **Post-Exploitation** – After getting access, I confirmed the root privileges, explored the system, checked users and processes, and located the flag.

---

## Tools Used

* **netdiscover** – Used to find devices on the local network
* **arp-scan** – Used to discover devices on the local network
* **Nmap** – Used for port scanning, service identification, and OS detection
* **Metasploit Framework** – Used to exploit the vulnerable Samba service
* **Linux command line** – Used during post-exploitation with commands such as bash, `cat`, `find`, `ps`, `netstat`, etc.

---

## Repository Structure

```
Kioptrix-Level1-Assessment/
├── 01-reconnaissance/                  # Discovery and scanning results
│   ├── network-discovery/              # netdiscover, arp-scan outputs
│   ├── port-scanning/                  # Nmap scans & open ports summary
│   ├── vulnerability-identification/   # Service version enumeration
│   └── screenshots/                    # Visual evidence (PNG files)
├── 02-security-assessment/             # Vulnerability analysis & exploitation
│   ├── vulnerability-analysis/         # Detailed write-ups for each vulnerability
│   ├── exploitation/                   # Metasploit session logs and commands
│   ├── post-exploitation/              # System exploration, user list, flag discovery
│   └── risk-assessment/                # Severity matrix and recommendations
├── 03-reporting/                       # Final deliverable
│   ├── executive-summary.md            # High-level overview for management
│   ├── technical-report.md             # Full technical details (this is the main report)
│   ├── evidence/                       # All screenshots and logs
│   └── findings/                       # Structured vulnerability findings
├── 04-tools/                           # (Optional) scripts or custom tools used
│   └── notes/                          # Raw notes taken during the engagement
├── README.md                           # This file
```

---

## How to Navigate

* **Start here** – this `README.md` gives a quick overview of what I did in the lab.
* **For the full technical story**, read: [`03-reporting/technical-report.md`](03-reporting/technical-report.md)
* **For a quick management summary**, see: [`03-reporting/executive-summary.md`](03-reporting/executive-summary.md)
* **For evidence**, browse the `03-reporting/evidence/` folder.
* **To reproduce the findings**, follow the steps in `02-security-assessment/exploitation/` – but **only in a lab environment!**

---

## Important Disclaimer

> ⚠️ **This is a vulnerable system.** The techniques and code shown here are for **educational purposes only**. Do not use them against any system without explicit written permission. The author is not responsible for any misuse or damage caused by this information.

---
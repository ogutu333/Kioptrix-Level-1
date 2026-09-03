# Attack Tree – Kioptrix Level 1

## Goal: Gain Root Access
[ GAIN ROOT ACCESS ]
│
┌───────────────┼───────────────┐
│ │ │
[Exploit SMB] [Exploit HTTPS] [Other]
│ │
▼ ▼
┌───────────┐ ┌─────────────┐
│ Samba │ │ mod_ssl │
│ trans2open│ │ buffer │
│ (CVE-2003-│ │ overflow │
│ 0201) │ │ (CVE-2002- │
│ │ │ 0082) │
└───────────┘ └─────────────┘
│ │
└───────┬───────┘
▼
[ ROOT SHELL ]

**Shortest path:** Samba trans2open (no authentication, immediate root) – **the path I chose.**
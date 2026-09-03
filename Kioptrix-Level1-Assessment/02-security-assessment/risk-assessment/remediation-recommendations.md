# Remediation Recommendations

## Priority 1 – Immediate (Within 24–48 Hours)

### R-01: Patch Samba
```bash
# Update to a patched version
yum update samba

# If patching is impossible, block SMB access
iptables -A INPUT -p tcp --dport 139 -j DROP
iptables -A INPUT -p tcp --dport 445 -j DROP
```

## Priority 2 – Within 1 Month

### R-02: Update Apache, mod_ssl, and OpenSSL
```bash
yum update httpd mod_ssl openssl
```

### R-03: Disable SSLv2/v3 (if still used)
In httpd.conf:

```text
SSLProtocol all -SSLv2 -SSLv3
```

### R-04: Upgrade OpenSSH and disable SSHv1
In /etc/ssh/sshd_config:

```text
Protocol 2
```

## Priority 3 – Within 3 Months

### R-05: Upgrade Linux Kernel or Migrate OS
- Install a modern Red Hat/CentOS version (7+).
- Or migrate to Ubuntu LTS with current kernel.

### R-06: Implement System Hardening
- Remove unnecessary services (xinetd, rpcbind, etc.).
- Remove unnecessary users (john, harold if not needed).
- Enable system logging and auditing.

---
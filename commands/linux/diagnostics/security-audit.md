# /linux-diagnostics-security-audit

Perform comprehensive security audit including sudo access, user accounts, SSH configuration, and authentication.

## Usage
`/linux-diagnostics-security-audit`
`/linux-diagnostics-security-audit --sudo-only`
`/linux-diagnostics-security-audit --ssh-only`
`/linux-diagnostics-security-audit --full`

## Tasks
1. Audit sudo access and sudoers configuration
2. Review user accounts and permissions
3. Check SSH configuration and security
4. Analyze authentication logs
5. Review file permissions on critical files
6. Check for security updates
7. Scan for suspicious activity
8. Generate security report

## Sudo Audit
```bash
# View sudoers file
cat /etc/sudoers

# List sudoers.d files
ls -la /etc/sudoers.d/

# Check all sudoers configurations
cat /etc/sudoers.d/*

# Check syntax
visudo -c

# List users with sudo access
grep -Po '^sudo.+:\K.*$' /etc/group
getent group sudo

# Users with NOPASSWD sudo
grep NOPASSWD /etc/sudoers /etc/sudoers.d/* 2>/dev/null

# Recent sudo usage
journalctl -t sudo -n 100
grep sudo /var/log/auth.log | tail -50

# Failed sudo attempts
grep "sudo.*authentication failure" /var/log/auth.log | tail -20

# Commands run with sudo
grep "COMMAND=" /var/log/auth.log | tail -30
```

## User Account Audit
```bash
# List all users
cat /etc/passwd

# Users with login shells
cat /etc/passwd | grep -v nologin | grep -v false

# Users with UID 0 (root privileges)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Users with empty passwords
awk -F: '$2 == "" {print $1}' /etc/shadow 2>/dev/null

# Recently created users (last 30 days)
awk -F: '{print $1}' /etc/passwd | while read user; do
  chage -l $user 2>/dev/null | grep "Last password change" | grep -v "never"
done

# Users who haven't changed password in 90+ days
for user in $(awk -F: '$7 !~ /nologin|false/ {print $1}' /etc/passwd); do
  chage -l $user 2>/dev/null
done

# Currently logged in users
who
w

# Last logins
last -20
lastlog | grep -v "Never"

# Failed login attempts
lastb -20 2>/dev/null || grep "Failed password" /var/log/auth.log | tail -20
```

## SSH Configuration Audit
```bash
# SSH server configuration
cat /etc/ssh/sshd_config

# Check critical settings
grep -E "^PermitRootLogin|^PasswordAuthentication|^PubkeyAuthentication|^Port" /etc/ssh/sshd_config

# Check for insecure settings
echo "=== Checking for security issues ==="
grep "^PermitRootLogin yes" /etc/ssh/sshd_config && echo "WARNING: Root login enabled"
grep "^PasswordAuthentication yes" /etc/ssh/sshd_config && echo "NOTICE: Password auth enabled"
grep "^PermitEmptyPasswords yes" /etc/ssh/sshd_config && echo "CRITICAL: Empty passwords allowed"
grep "^X11Forwarding yes" /etc/ssh/sshd_config && echo "INFO: X11 forwarding enabled"

# Test SSH config syntax
sshd -t

# Active SSH sessions
ss -tnp | grep :22
who | grep pts

# SSH authentication logs
grep sshd /var/log/auth.log | tail -50

# Failed SSH attempts
grep "Failed password" /var/log/auth.log | tail -30
grep "Invalid user" /var/log/auth.log | tail -30

# Successful SSH logins
grep "Accepted publickey\|Accepted password" /var/log/auth.log | tail -20

# SSH brute force attempts (IPs)
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -10
```

## Authorized Keys Audit
```bash
# Check root's authorized keys
ls -la /root/.ssh/
cat /root/.ssh/authorized_keys 2>/dev/null

# Check all users' authorized keys
for home in /home/*; do
  user=$(basename $home)
  if [ -f "$home/.ssh/authorized_keys" ]; then
    echo "=== $user ==="
    ls -la $home/.ssh/authorized_keys
    cat $home/.ssh/authorized_keys
  fi
done

# Find all SSH keys on system
find /home -name "authorized_keys" -o -name "id_rsa.pub" -o -name "id_ed25519.pub" 2>/dev/null
```

## Authentication and PAM
```bash
# PAM configuration for SSH
cat /etc/pam.d/sshd

# PAM common auth
cat /etc/pam.d/common-auth

# Check for fail2ban
systemctl status fail2ban
fail2ban-client status
fail2ban-client status sshd

# Account lockout settings
grep pam_tally2 /etc/pam.d/* 2>/dev/null
grep pam_faillock /etc/pam.d/* 2>/dev/null
```

## Critical File Permissions
```bash
# Check permissions on sensitive files
ls -l /etc/passwd /etc/shadow /etc/group /etc/gshadow

# Sudoers permissions
ls -l /etc/sudoers /etc/sudoers.d/

# SSH config permissions
ls -l /etc/ssh/sshd_config

# Find world-writable files in /etc
find /etc -type f -perm -002 -ls 2>/dev/null

# Find SUID/SGID files
find / -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null | head -50

# Check for files with no owner
find / -nouser -o -nogroup 2>/dev/null | head -20
```

## Security Updates
```bash
# Debian/Ubuntu
apt list --upgradable 2>/dev/null | grep -i security
apt-get upgrade --dry-run | grep -i security

# RHEL/CentOS
yum list updates --security 2>/dev/null
dnf updateinfo list --security 2>/dev/null

# Check unattended-upgrades (Ubuntu/Debian)
systemctl status unattended-upgrades
cat /etc/apt/apt.conf.d/50unattended-upgrades 2>/dev/null
```

## Suspicious Activity Detection
```bash
# Check for unusual cron jobs
crontab -l
ls -la /etc/cron.*
cat /etc/crontab

# System-wide cron jobs
for user in $(cut -f1 -d: /etc/passwd); do
  echo "=== Cron for $user ==="
  crontab -u $user -l 2>/dev/null
done

# Recently modified files in /tmp
find /tmp -type f -mtime -1 -ls 2>/dev/null | head -20

# Listening ports and services
ss -tlnp
netstat -tlnp

# Unusual processes
ps aux --sort=-%mem | head -20
ps aux --sort=-%cpu | head -20

# Check for rootkits (if rkhunter installed)
rkhunter --check --skip-keypress --report-warnings-only 2>/dev/null || echo "rkhunter not installed"

# Check for suspicious network connections
ss -tnp | grep ESTABLISHED
```

## Password Policy
```bash
# Password aging settings
cat /etc/login.defs | grep -E "^PASS_MAX_DAYS|^PASS_MIN_DAYS|^PASS_WARN_AGE"

# Check password quality requirements
cat /etc/security/pwquality.conf 2>/dev/null
grep pam_pwquality /etc/pam.d/* 2>/dev/null
```

## Firewall Status
```bash
# UFW (Ubuntu)
ufw status verbose

# firewalld (RHEL/CentOS)
firewall-cmd --list-all
firewall-cmd --list-services

# iptables
iptables -L -n -v
iptables -L INPUT -n -v
iptables -L OUTPUT -n -v

# Check for open ports
ss -tlnp
nmap -sT localhost 2>/dev/null || netstat -tlnp
```

## SELinux/AppArmor
```bash
# SELinux status (RHEL/CentOS)
getenforce 2>/dev/null
sestatus 2>/dev/null
ausearch -m avc -ts recent 2>/dev/null | head -20

# AppArmor status (Ubuntu/Debian)
aa-status 2>/dev/null
systemctl status apparmor
```

## Audit Logs
```bash
# Check if auditd is running
systemctl status auditd

# Recent audit events
ausearch -ts today 2>/dev/null | head -50

# Login events
ausearch -m USER_LOGIN -ts today 2>/dev/null

# Failed login events
ausearch -m USER_AUTH -sv no -ts today 2>/dev/null
```

## Quick Security Hardening
```bash
# Disable root SSH login (backup first)
sed -i.bak 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl reload sshd

# Enforce key-based auth only
sed -i 's/^PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
systemctl reload sshd

# Install fail2ban
apt install fail2ban -y  # Debian/Ubuntu
yum install fail2ban -y  # RHEL/CentOS
systemctl enable --now fail2ban

# Enable automatic security updates (Ubuntu/Debian)
apt install unattended-upgrades -y
dpkg-reconfigure -plow unattended-upgrades
```

## Output Format
Provide a security audit report with:
- Critical issues (immediate action required)
  - Root login enabled
  - Users with NOPASSWD sudo
  - Users with empty passwords
  - World-writable sensitive files
- Warnings (should be addressed)
  - Password authentication enabled
  - Missing security updates
  - Weak password policies
  - Unusual SUID binaries
- Information
  - Current sudo users
  - Active SSH sessions
  - Recent failed logins
  - Firewall status
- Recommendations
  - Hardening steps
  - Configuration improvements
  - Monitoring suggestions

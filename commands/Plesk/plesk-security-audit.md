# /plesk-security-audit

Perform a security audit on Plesk server or specific domain.

## Usage
`/plesk-security-audit`
`/plesk-security-audit example.com`
`/plesk-security-audit --full`

## Tasks
1. Check Plesk and component versions
2. Audit SSL/TLS configuration
3. Check for outdated software
4. Review firewall rules
5. Check for weak passwords (policies)
6. Scan for malware/suspicious files
7. Review fail2ban status
8. Generate security report

## Server-Level Checks
```bash
# Check Plesk version
plesk version

# Check for available updates
plesk installer --select-release-current --show-components 2>/dev/null | head -20

# List admin sessions
plesk bin admin --list-sessions

# Check Plesk security settings
plesk bin server_pref --show | grep -i security

# Fail2Ban status
plesk bin ip_ban --status
fail2ban-client status

# ModSecurity status
plesk bin server_pref --show | grep -i modsecurity

# Check open ports
ss -tlnp | grep LISTEN

# Recent failed logins
grep "Failed password" /var/log/auth.log | tail -20
```

## SSL/TLS Audit
```bash
# Check Plesk panel certificate
openssl s_client -connect localhost:8443 -servername $(hostname) 2>/dev/null | openssl x509 -noout -dates

# List all SSL certificates and expiry
for domain in $(plesk bin domain --list); do
  echo "=== $domain ==="
  plesk bin certificate --list -domain $domain 2>/dev/null | head -5
done

# Test SSL configuration
plesk bin certificate --info -domain example.com

# Check for weak ciphers (external test)
nmap --script ssl-enum-ciphers -p 443 example.com
```

## Domain-Level Audit
```bash
# Check domain security settings
plesk bin domain --info example.com

# List FTP accounts
plesk bin ftpsubaccount --list example.com

# Check for suspicious cron jobs
crontab -u $(plesk bin domain --info example.com | grep "Owner" | awk '{print $2}') -l

# Scan for malicious files
clamscan -r /var/www/vhosts/example.com/httpdocs/ 2>/dev/null || echo "ClamAV not installed"

# Find recently modified PHP files
find /var/www/vhosts/example.com/httpdocs -name "*.php" -mtime -7 -ls

# Check file permissions
find /var/www/vhosts/example.com/httpdocs -type f -perm 0777 -ls
find /var/www/vhosts/example.com/httpdocs -type d -perm 0777 -ls

# Check .htaccess for suspicious rules
find /var/www/vhosts/example.com -name ".htaccess" -exec grep -l "base64\|eval\|exec" {} \;
```

## Password Policy
```bash
# Check password strength requirements
plesk bin server_pref --show | grep -i password

# Set strong password policy
plesk bin server_pref --update -min_password_strength very_strong
```

## Firewall Review
```bash
# Plesk Firewall status
plesk bin firewall --show

# IPTables rules
iptables -L -n -v | head -50

# UFW status (if used)
ufw status verbose 2>/dev/null
```

## Quick Fixes
```bash
# Enable fail2ban for Plesk
plesk bin ip_ban --enable

# Update ModSecurity rules
plesk bin modsecurity --update

# Force password change for all users
# (use cautiously)
plesk bin admin --set-admin-password -passwd "$(openssl rand -base64 16)"

# Disable unused services
plesk bin service --disable courier-imap  # if not using
```

## Output Format
Generate a security report with:
- Critical issues (immediate action required)
- Warnings (should be addressed)
- Recommendations (best practices)
- Current security score/status
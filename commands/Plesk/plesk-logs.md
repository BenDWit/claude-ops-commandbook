# /plesk-logs

View and analyze Plesk logs for troubleshooting.

## Usage
`/plesk-logs example.com`
`/plesk-logs example.com --error`
`/plesk-logs example.com --access --tail 100`
`/plesk-logs --panel`
`/plesk-logs --mail`

## Tasks
1. Identify relevant log files
2. Display recent entries
3. Filter for errors/warnings
4. Analyze patterns
5. Suggest solutions for common errors

## Domain Logs
```bash
# Error log
tail -100 /var/www/vhosts/example.com/logs/error_log

# Access log
tail -100 /var/www/vhosts/example.com/logs/access_log

# SSL access log
tail -100 /var/www/vhosts/example.com/logs/access_ssl_log

# PHP-FPM errors (domain specific)
grep "example.com" /var/log/plesk-php*-fpm/error.log | tail -50

# Recent errors only
grep -i "error\|fatal\|critical" /var/www/vhosts/example.com/logs/error_log | tail -50

# 500 errors in access log
grep '" 500 ' /var/www/vhosts/example.com/logs/access_log | tail -20
```

## Plesk Panel Logs
```bash
# Main Plesk log
tail -100 /var/log/plesk/panel.log

# Plesk UI errors
grep -i "error" /var/log/plesk/panel.log | tail -30

# API actions
grep -i "api" /var/log/plesk/panel.log | tail -30

# Extension logs
ls -la /var/log/plesk/ext-*.log
tail -50 /var/log/plesk/ext-letsencrypt.log

# Backup logs
tail -100 /var/log/plesk/pmmcli.log
```

## Web Server Logs
```bash
# Apache error log
tail -100 /var/log/apache2/error_log

# Apache access log (global)
tail -100 /var/log/apache2/access_log

# Nginx error log
tail -100 /var/log/nginx/error.log

# Nginx access log
tail -100 /var/log/nginx/access.log

# Recent 404s
grep '" 404 ' /var/log/apache2/access_log | tail -20
grep '" 404 ' /var/log/nginx/access.log | tail -20
```

## Mail Logs
```bash
# Main mail log
tail -100 /var/log/maillog

# Failed deliveries
grep -i "status=bounced" /var/log/maillog | tail -20

# Authentication failures
grep -i "auth.*fail\|login.*fail" /var/log/maillog | tail -20

# Spam/rejected
grep -i "reject\|spam" /var/log/maillog | tail -20
```

## Database Logs
```bash
# MySQL/MariaDB error log
tail -100 /var/log/mysql/error.log

# Slow queries
tail -50 /var/log/mysql/slow.log
```

## Security Logs
```bash
# Fail2ban
tail -100 /var/log/fail2ban.log
fail2ban-client status

# SSH auth
grep -i "failed\|accepted" /var/log/auth.log | tail -50

# ModSecurity (if enabled)
tail -100 /var/log/modsec_audit.log
```

## Analysis Commands
```bash
# Top IPs by requests
awk '{print $1}' /var/www/vhosts/example.com/logs/access_log | sort | uniq -c | sort -rn | head -20

# Top URLs requested
awk '{print $7}' /var/www/vhosts/example.com/logs/access_log | sort | uniq -c | sort -rn | head -20

# Top user agents
awk -F'"' '{print $6}' /var/www/vhosts/example.com/logs/access_log | sort | uniq -c | sort -rn | head -20

# Requests per hour
awk '{print $4}' /var/www/vhosts/example.com/logs/access_log | cut -d: -f1,2 | sort | uniq -c | tail -24

# Error types breakdown
awk '{print $NF}' /var/www/vhosts/example.com/logs/error_log | sort | uniq -c | sort -rn | head -20
```

## Log Rotation
```bash
# Check log rotation config
cat /etc/logrotate.d/plesk

# Force rotation
logrotate -f /etc/logrotate.d/plesk

# Check disk usage by logs
du -sh /var/log/*
du -sh /var/www/vhosts/*/logs/
```

## Real-time Monitoring
```bash
# Follow error log
tail -f /var/www/vhosts/example.com/logs/error_log

# Follow access log with filter
tail -f /var/www/vhosts/example.com/logs/access_log | grep -v "200\|304"

# Follow multiple logs
tail -f /var/www/vhosts/example.com/logs/error_log /var/log/apache2/error_log
```
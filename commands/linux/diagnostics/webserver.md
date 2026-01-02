# /linux-diagnostics-webserver

Diagnose Apache and Nginx webserver issues including logs, configuration, and performance.

## Usage
`/linux-diagnostics-webserver`
`/linux-diagnostics-webserver --nginx`
`/linux-diagnostics-webserver --apache`
`/linux-diagnostics-webserver --errors-only`

## Tasks
1. Detect which webserver is running (Apache/Nginx/both)
2. Check webserver service status
3. Analyze access and error logs
4. Review configuration syntax
5. Check virtual hosts/server blocks
6. Analyze connection limits and performance
7. Check SSL/TLS configuration
8. Generate diagnostic report

## Service Detection and Status
```bash
# Check which webserver is installed/running
systemctl status apache2 2>/dev/null || systemctl status httpd 2>/dev/null
systemctl status nginx

# Check if listening on ports
ss -tlnp | grep -E ':80|:443'
netstat -tlnp | grep -E ':80|:443'

# Check processes
ps aux | grep -E 'apache2|httpd|nginx' | grep -v grep

# Version information
apache2 -v 2>/dev/null || httpd -v 2>/dev/null
nginx -v 2>/dev/null
```

## Apache Diagnostics
```bash
# Service status
systemctl status apache2  # Debian/Ubuntu
systemctl status httpd    # RHEL/CentOS

# Configuration test
apache2ctl -t  # Debian/Ubuntu
httpd -t       # RHEL/CentOS

# Show loaded modules
apache2ctl -M  # Debian/Ubuntu
httpd -M       # RHEL/CentOS

# Show virtual hosts
apache2ctl -S  # Debian/Ubuntu
httpd -S       # RHEL/CentOS

# Check configuration files
cat /etc/apache2/apache2.conf  # Debian/Ubuntu
cat /etc/httpd/conf/httpd.conf # RHEL/CentOS

# List enabled sites
ls -l /etc/apache2/sites-enabled/  # Debian/Ubuntu
ls -l /etc/httpd/conf.d/           # RHEL/CentOS
```

## Nginx Diagnostics
```bash
# Service status
systemctl status nginx

# Configuration test
nginx -t

# Show configuration
nginx -T

# Main config file
cat /etc/nginx/nginx.conf

# List server blocks
ls -l /etc/nginx/sites-enabled/  # Debian/Ubuntu
ls -l /etc/nginx/conf.d/         # RHEL/CentOS

# Check included configs
grep -r "include" /etc/nginx/nginx.conf
```

## Apache Log Analysis
```bash
# Error log locations
# Debian/Ubuntu: /var/log/apache2/error.log
# RHEL/CentOS: /var/log/httpd/error_log

# Recent errors
tail -100 /var/log/apache2/error.log
tail -100 /var/log/httpd/error_log

# Follow error log
tail -f /var/log/apache2/error.log

# Count error types
grep "\[error\]" /var/log/apache2/error.log | wc -l
grep "\[crit\]" /var/log/apache2/error.log | wc -l

# Search for specific errors
grep -i "permission denied" /var/log/apache2/error.log
grep -i "no such file" /var/log/apache2/error.log
grep -i "timeout" /var/log/apache2/error.log

# Access log analysis
tail -100 /var/log/apache2/access.log

# Top IP addresses
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20

# Top requested URLs
awk '{print $7}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20

# HTTP status codes
awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -rn

# 404 errors
grep " 404 " /var/log/apache2/access.log | tail -20

# 500 errors
grep " 500 " /var/log/apache2/access.log | tail -20

# Requests per minute (last 1000 lines)
tail -1000 /var/log/apache2/access.log | awk '{print $4}' | cut -d: -f1-2 | uniq -c
```

## Nginx Log Analysis
```bash
# Error log
tail -100 /var/log/nginx/error.log

# Follow error log
tail -f /var/log/nginx/error.log

# Count error levels
grep "\[error\]" /var/log/nginx/error.log | wc -l
grep "\[crit\]" /var/log/nginx/error.log | wc -l
grep "\[alert\]" /var/log/nginx/error.log | wc -l

# Access log
tail -100 /var/log/nginx/access.log

# Top IPs
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Top URLs
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -20

# Status codes
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# Upstream errors
grep "upstream" /var/log/nginx/error.log | tail -20

# Connection refused errors
grep "connect() failed" /var/log/nginx/error.log | tail -20
```

## Performance Metrics
```bash
# Apache status (requires mod_status)
curl http://localhost/server-status
curl http://localhost/server-status?auto

# Apache worker/connection info
apache2ctl status  # Debian/Ubuntu
httpd status       # RHEL/CentOS

# Nginx stub status (if configured)
curl http://localhost/nginx_status

# Current connections
ss -ant | grep -E ':80|:443' | wc -l

# Apache processes
ps aux | grep apache2 | wc -l
ps aux | grep httpd | wc -l

# Nginx processes
ps aux | grep nginx | wc -l

# Memory usage
ps aux | grep -E 'apache2|httpd' | awk '{sum+=$6} END {print "Apache Memory: " sum/1024 " MB"}'
ps aux | grep nginx | awk '{sum+=$6} END {print "Nginx Memory: " sum/1024 " MB"}'
```

## Connection Limits
```bash
# Apache MPM configuration
apachectl -M | grep mpm
grep -r "MaxRequestWorkers\|MaxClients" /etc/apache2/  # Debian/Ubuntu
grep -r "MaxRequestWorkers\|MaxClients" /etc/httpd/   # RHEL/CentOS

# Nginx worker processes and connections
grep worker /etc/nginx/nginx.conf
```

## SSL/TLS Configuration
```bash
# Apache SSL modules
apache2ctl -M | grep ssl
httpd -M | grep ssl

# Apache SSL config
grep -r "SSLEngine\|SSLCertificate" /etc/apache2/sites-enabled/
grep -r "SSLEngine\|SSLCertificate" /etc/httpd/conf.d/

# Nginx SSL config
grep -r "ssl_certificate" /etc/nginx/

# Test SSL certificate
openssl s_client -connect localhost:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -dates

# Check certificate expiry for all configured sites
for cert in $(grep -r "ssl_certificate " /etc/nginx/ | awk '{print $2}' | tr -d ';' | sort -u); do
  echo "=== $cert ==="
  openssl x509 -in $cert -noout -dates 2>/dev/null
done
```

## Common Issues
```bash
# Check for syntax errors
apache2ctl configtest  # Apache Debian/Ubuntu
httpd -t              # Apache RHEL/CentOS
nginx -t              # Nginx

# File permissions
ls -la /var/www/html/
namei -l /var/www/html/index.html

# SELinux issues (RHEL/CentOS)
getenforce
ausearch -m avc -ts recent | grep httpd

# Check firewall
ufw status | grep -E "80|443"  # Ubuntu
firewall-cmd --list-all        # RHEL/CentOS

# Disk space (logs can fill up)
df -h /var/log

# Open file limits
ulimit -n
cat /proc/$(pgrep nginx | head -1)/limits | grep "open files"
```

## PHP-FPM Integration (if applicable)
```bash
# PHP-FPM status
systemctl status php*-fpm
systemctl status php-fpm

# PHP-FPM pool configuration
ls -la /etc/php/*/fpm/pool.d/
cat /etc/php/*/fpm/pool.d/www.conf

# PHP-FPM logs
tail -50 /var/log/php*-fpm.log

# PHP-FPM socket/port
ss -tlnp | grep php-fpm
ls -la /var/run/php/
```

## Quick Fixes
```bash
# Reload configuration (graceful)
systemctl reload apache2  # or httpd
systemctl reload nginx

# Restart service
systemctl restart apache2  # or httpd
systemctl restart nginx

# Enable service on boot
systemctl enable apache2  # or httpd
systemctl enable nginx

# Rotate logs manually
logrotate -f /etc/logrotate.d/apache2
logrotate -f /etc/logrotate.d/nginx
```

## Output Format
Provide a diagnostic report with:
- Webserver type and version
- Service status (running/stopped/errors)
- Configuration syntax status (valid/errors)
- Recent critical errors from logs
- Performance metrics (connections, memory)
- SSL certificate status and expiry
- Top traffic sources and URLs
- Recommendations for issues found

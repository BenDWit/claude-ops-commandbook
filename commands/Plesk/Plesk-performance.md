# /plesk-perf

Analyze performance issues on Plesk server or specific domain.

## Usage
`/plesk-perf`
`/plesk-perf example.com`
`/plesk-perf --resources`
`/plesk-perf --slow-queries`

## Tasks
1. Check system resources (CPU, RAM, disk)
2. Identify resource-hungry processes
3. Check web server performance
4. Analyze database performance
5. Review PHP-FPM pools
6. Generate optimization recommendations

## System Resources
```bash
# Overall system status
top -bn1 | head -20

# CPU info
nproc
uptime
mpstat 1 5

# Memory usage
free -h
vmstat 1 5

# Disk usage
df -h
iostat -x 1 5

# Disk I/O heavy processes
iotop -bn1 | head -20
```

## Process Analysis
```bash
# Top CPU consumers
ps aux --sort=-%cpu | head -15

# Top memory consumers
ps aux --sort=-%mem | head -15

# Apache/Nginx connections
ss -s
ss -tlnp | grep -E "80|443"

# Current connections per IP
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -20
```

## Web Server Performance
```bash
# Apache status (if mod_status enabled)
curl -s http://localhost/server-status?auto

# Apache processes
ps aux | grep apache | wc -l

# Nginx connections
curl -s http://localhost/nginx_status

# Check Apache config for issues
apache2ctl -t
apache2ctl -M | wc -l

# Nginx config test
nginx -t

# Check for mod_pagespeed or similar
apache2ctl -M | grep -i "pagespeed\|cache\|deflate"
```

## PHP-FPM Analysis
```bash
# List PHP-FPM pools
ls /etc/php-fpm.d/ 2>/dev/null || ls /etc/php/*/fpm/pool.d/

# PHP-FPM status
systemctl status plesk-php*-fpm

# PHP-FPM processes
ps aux | grep php-fpm | wc -l

# PHP-FPM pool status (if enabled)
curl -s "http://localhost/fpm-status?full"

# Check pool configuration
grep -r "pm.max_children\|pm.start_servers" /etc/php-fpm.d/

# Domain-specific PHP settings
plesk bin site --info example.com -php
```

## Database Performance
```bash
# MySQL process list
plesk db "SHOW PROCESSLIST;"

# Slow queries
plesk db "SHOW GLOBAL STATUS LIKE 'Slow_queries';"

# Current connections
plesk db "SHOW STATUS LIKE 'Threads_connected';"

# Max connections
plesk db "SHOW VARIABLES LIKE 'max_connections';"

# Buffer pool usage
plesk db "SHOW STATUS LIKE 'Innodb_buffer_pool%';"

# Table lock waits
plesk db "SHOW STATUS LIKE '%lock%';"

# Recent slow queries
tail -50 /var/log/mysql/slow.log
```

## Domain-Specific Performance
```bash
# Request processing time (last 100 requests)
awk '{print $NF}' /var/www/vhosts/example.com/logs/access_log | tail -100 | sort -n

# Large files being served
awk '{print $10, $7}' /var/www/vhosts/example.com/logs/access_log | sort -rn | head -20

# PHP scripts causing errors
grep "PHP" /var/www/vhosts/example.com/logs/error_log | tail -20

# Check for heavy cron jobs
crontab -u $(stat -c '%U' /var/www/vhosts/example.com/) -l 2>/dev/null
```

## Quick Optimizations
```bash
# Increase PHP memory for domain
plesk bin site --update-php-settings example.com -settings "memory_limit=512M"

# Enable OPcache (verify it's on)
php -i | grep opcache

# Enable gzip compression (Apache)
a2enmod deflate

# Restart services
systemctl restart apache2 nginx plesk-php82-fpm

# Clear Plesk cache
plesk repair web -y

# MySQL optimization
mysqlcheck -u admin -p$(cat /etc/psa/.psa.shadow) --optimize --all-databases
```

## Monitoring Commands
```bash
# Real-time resource monitoring
htop

# Network connections real-time
watch -n1 'ss -s'

# Disk I/O real-time
iotop

# MySQL real-time
watch -n1 'mysql -u admin -p$(cat /etc/psa/.psa.shadow) -e "SHOW PROCESSLIST"'
```

## Benchmark Tools
```bash
# Quick HTTP benchmark
ab -n 100 -c 10 https://example.com/

# More detailed benchmark
siege -c 10 -t 30s https://example.com/

# PHP benchmark
php -r "echo 'PHP execution: ', microtime(true) - \$_SERVER['REQUEST_TIME_FLOAT'], 's';"
```

## Output Format
Provide performance report with:
- System load (1/5/15 min averages)
- Memory usage (used/available/swap)
- Disk usage per partition
- Top 5 resource consumers
- Database connection count
- Web server connection count
- Identified bottlenecks
- Recommended optimizations
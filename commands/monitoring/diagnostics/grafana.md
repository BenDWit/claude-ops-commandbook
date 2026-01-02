# /monitoring-diagnostics-grafana

Diagnose Grafana service health, datasource connectivity, dashboard issues, and performance metrics.

## Usage
`/monitoring-diagnostics-grafana`
`/monitoring-diagnostics-grafana --service`
`/monitoring-diagnostics-grafana --datasources`
`/monitoring-diagnostics-grafana --dashboards`
`/monitoring-diagnostics-grafana --performance`

## Tasks
1. Check Grafana service status and health
2. Verify datasource connectivity
3. Review dashboard and panel errors
4. Check alert rule status and notifications
5. Analyze performance metrics
6. Review API and database connectivity
7. Check authentication and authorization
8. Generate comprehensive diagnostic report

## Service Status & Detection
```bash
# Check Grafana service (native installation)
systemctl status grafana-server

# Docker container check
docker ps | grep grafana
docker ps -a | grep grafana  # Include stopped containers

# Docker Compose
docker-compose ps grafana 2>/dev/null

# Process check
ps aux | grep grafana | grep -v grep

# Listening ports
ss -tlnp | grep :3000
netstat -tlnp | grep :3000

# Grafana version (native)
grafana-cli --version
grafana-server -v 2>/dev/null

# Grafana version (Docker)
docker exec grafana grafana-cli --version 2>/dev/null

# Process uptime and resource usage
ps -eo pid,etime,cmd,%cpu,%mem | grep grafana | grep -v grep

# Service enabled on boot
systemctl is-enabled grafana-server 2>/dev/null
```

## Grafana Health Check
```bash
# API health endpoint
curl -s http://localhost:3000/api/health
curl -s http://localhost:3000/api/health | jq

# With authentication
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" http://localhost:3000/api/health | jq

# Database connectivity test
curl -s http://localhost:3000/api/admin/stats 2>/dev/null | jq

# Check if Grafana is responding
timeout 5 curl -s http://localhost:3000/login > /dev/null && echo "Grafana responding" || echo "Grafana not responding"

# Service logs - native installation
tail -100 /var/log/grafana/grafana.log
journalctl -u grafana-server -n 100

# Service logs - Docker
docker logs grafana --tail 100
docker logs grafana --tail 100 2>&1 | grep -E "error|warn|fail"

# Check for startup errors
journalctl -u grafana-server | grep -i error | tail -20
docker logs grafana 2>&1 | grep -i error | tail -20

# Service restart count
journalctl -u grafana-server | grep -c "Started Grafana"
```

## Configuration Review
```bash
# Configuration file - native
cat /etc/grafana/grafana.ini
grep -v "^#\|^;" /etc/grafana/grafana.ini | grep -v "^$"  # Without comments

# Configuration file - Docker
docker exec grafana cat /etc/grafana/grafana.ini
docker exec grafana cat /usr/share/grafana/conf/defaults.ini

# Data directory
ls -lah /var/lib/grafana/
du -sh /var/lib/grafana/

# Docker volumes
docker inspect grafana | jq '.[0].Mounts'

# Database type from config
grep "^type =" /etc/grafana/grafana.ini 2>/dev/null || echo "Using default (sqlite3)"

# Check environment variables (Docker)
docker exec grafana env | grep ^GF_

# Plugin directory
ls -lah /var/lib/grafana/plugins/ 2>/dev/null
docker exec grafana ls -lah /var/lib/grafana/plugins/ 2>/dev/null

# List installed plugins
grafana-cli plugins ls 2>/dev/null
docker exec grafana grafana-cli plugins ls 2>/dev/null

# Provisioning configuration
ls -lah /etc/grafana/provisioning/datasources/
ls -lah /etc/grafana/provisioning/dashboards/
```

## Datasource Health & Connectivity
```bash
# List all datasources (requires API token)
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/datasources | jq

# Get datasource by ID
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/datasources/1 | jq

# Test datasource health
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/datasources/uid/{datasource-uid}/health | jq

# Test all datasources
for ds_id in $(curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/datasources | jq -r '.[].id'); do
  echo "Testing datasource ID: $ds_id"
  curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
    "http://localhost:3000/api/datasources/$ds_id/health" | jq '.message'
done

# Check Prometheus datasource connectivity
curl -s http://prometheus:9090/api/v1/status/config 2>/dev/null && echo "Prometheus reachable"

# Check InfluxDB connectivity (if configured)
curl -s http://influxdb:8086/health 2>/dev/null && echo "InfluxDB reachable"

# Network connectivity to common datasources
nc -zv prometheus 9090 2>&1
nc -zv influxdb 8086 2>&1
nc -zv mysql 3306 2>&1

# Check datasource proxy settings in logs
grep "datasource proxy" /var/log/grafana/grafana.log | tail -20
```

## Dashboard Diagnostics
```bash
# List all dashboards
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/search | jq

# Count dashboards
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/search?type=dash-db | jq 'length'

# Search for specific dashboard
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  "http://localhost:3000/api/search?query=system" | jq

# Get dashboard by UID
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/dashboards/uid/{dashboard-uid} | jq

# Dashboard errors from logs
grep "dashboard.*error" /var/log/grafana/grafana.log | tail -30
docker logs grafana 2>&1 | grep "dashboard.*error" | tail -30

# Panel query errors
grep "panel query error" /var/log/grafana/grafana.log | tail -20

# Recently modified dashboards
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  "http://localhost:3000/api/search?type=dash-db" | \
  jq -r 'sort_by(.updated) | reverse | .[0:5] | .[] | [.title, .updated] | @tsv'

# Dashboard versions
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/dashboards/id/{dashboard-id}/versions | jq

# Dashboard permissions
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/dashboards/id/{dashboard-id}/permissions | jq
```

## Alert Rule Status
```bash
# List all alert rules (Grafana 8+)
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/ruler/grafana/api/v1/rules | jq

# Get alert state
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/prometheus/grafana/api/v1/alerts | jq

# Legacy alerting (Grafana <8)
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/alerts | jq

# Notification channels
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/alert-notifications | jq

# Alert history from logs
grep "alert" /var/log/grafana/grafana.log | grep -E "firing|resolved" | tail -30

# Failed alerts
grep "alert.*failed\|alert.*error" /var/log/grafana/grafana.log | tail -20

# Notification delivery status
grep "notification" /var/log/grafana/grafana.log | tail -20

# Contact points (Grafana 8+)
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/v1/provisioning/contact-points | jq

# Notification policies
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/v1/provisioning/policies | jq
```

## Performance Metrics
```bash
# Process CPU and memory
ps aux | grep grafana | grep -v grep
top -bn1 | grep grafana

# Detailed process stats
pidstat -p $(pgrep grafana-server) 1 5 2>/dev/null

# Docker container stats
docker stats grafana --no-stream

# API response time test
time curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/health

# Detailed timing with curl
curl -w "@-" -o /dev/null -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/health <<'EOF'
\ntime_total: %{time_total}s
time_connect: %{time_connect}s
time_starttransfer: %{time_starttransfer}s
EOF

# Query performance from logs
grep "query.*took" /var/log/grafana/grafana.log | tail -20

# Slow queries
grep "query took.*[5-9][0-9][0-9]ms\|query took.*[0-9]s" /var/log/grafana/grafana.log | tail -20

# Database query performance
grep "db query" /var/log/grafana/grafana.log | tail -20

# Panel render times
grep "rendering.*took" /var/log/grafana/grafana.log | tail -20

# Cache statistics
grep "cache" /var/log/grafana/grafana.log | tail -20
```

## Database Backend
```bash
# SQLite database (default)
ls -lh /var/lib/grafana/grafana.db
du -h /var/lib/grafana/grafana.db

# SQLite tables
sqlite3 /var/lib/grafana/grafana.db ".tables" 2>/dev/null

# SQLite integrity check
sqlite3 /var/lib/grafana/grafana.db "PRAGMA integrity_check;" 2>/dev/null

# PostgreSQL connection test
PGPASSWORD=grafana psql -h localhost -U grafana -d grafana -c "SELECT version();" 2>/dev/null

# PostgreSQL tables
PGPASSWORD=grafana psql -h localhost -U grafana -d grafana -c "\dt" 2>/dev/null

# PostgreSQL database size
PGPASSWORD=grafana psql -h localhost -U grafana -d grafana -c \
  "SELECT pg_size_pretty(pg_database_size('grafana'));" 2>/dev/null

# MySQL connection test
mysql -h localhost -u grafana -pgrafana -e "SELECT VERSION();" 2>/dev/null

# MySQL tables
mysql -h localhost -u grafana -pgrafana grafana -e "SHOW TABLES;" 2>/dev/null

# MySQL database size
mysql -h localhost -u grafana -pgrafana -e \
  "SELECT table_schema 'Database', ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) 'Size (MB)' \
   FROM information_schema.tables WHERE table_schema='grafana';" 2>/dev/null

# Database connection errors
grep "database.*error\|db.*connection" /var/log/grafana/grafana.log | tail -20

# Database locks
grep "database.*locked\|database.*busy" /var/log/grafana/grafana.log | tail -20
```

## Authentication & Users
```bash
# List users
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/org/users | jq

# Current user
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/user | jq

# Active sessions (admin API)
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/admin/users | jq

# LDAP configuration check
grep -A 10 "\[auth.ldap\]" /etc/grafana/grafana.ini

# OAuth configuration
grep -A 10 "\[auth" /etc/grafana/grafana.ini | grep -v "^#"

# Authentication failures from logs
grep "authentication.*failed\|login.*failed" /var/log/grafana/grafana.log | tail -30

# API token usage
grep "api.*token" /var/log/grafana/grafana.log | tail -20

# User permissions and roles
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/user/orgs | jq

# Organizations
curl -s -H "Authorization: Bearer $GRAFANA_TOKEN" \
  http://localhost:3000/api/orgs | jq
```

## Plugin Status
```bash
# List installed plugins
grafana-cli plugins ls 2>/dev/null
docker exec grafana grafana-cli plugins ls 2>/dev/null

# Plugin directory
ls -lah /var/lib/grafana/plugins/

# Plugin versions
for plugin in /var/lib/grafana/plugins/*; do
  echo "=== $(basename $plugin) ==="
  cat $plugin/plugin.json 2>/dev/null | jq -r '.info.version'
done

# Plugin errors from logs
grep "plugin.*error\|plugin.*failed" /var/log/grafana/grafana.log | tail -30

# Plugin loading
grep "plugin.*loaded\|plugin.*initialized" /var/log/grafana/grafana.log | tail -20

# Check for outdated plugins
grep "plugin.*update" /var/log/grafana/grafana.log | tail -10

# Panel plugins vs datasource plugins
grafana-cli plugins ls 2>/dev/null | grep -E "panel|datasource"
```

## Network & Proxy
```bash
# Listening ports
ss -tlnp | grep grafana
netstat -tlnp | grep grafana

# Grafana HTTP settings
grep -E "^http_addr|^http_port|^protocol" /etc/grafana/grafana.ini

# SSL/TLS certificate (if configured)
openssl s_client -connect localhost:3000 -servername localhost </dev/null 2>/dev/null | \
  openssl x509 -noout -dates

# Reverse proxy headers
curl -I http://localhost:3000/login

# CORS configuration
grep -A 5 "cors" /etc/grafana/grafana.ini

# Proxy settings
grep -A 5 "\[proxy\]" /etc/grafana/grafana.ini

# Network connectivity to datasources
for host in prometheus:9090 influxdb:8086 mysql:3306; do
  timeout 2 bash -c "cat < /dev/null > /dev/tcp/${host%:*}/${host#*:}" 2>/dev/null && \
    echo "$host: reachable" || echo "$host: unreachable"
done
```

## Log Analysis
```bash
# Recent errors (native)
tail -100 /var/log/grafana/grafana.log | grep -i error

# Recent errors (Docker)
docker logs grafana --tail 100 2>&1 | grep -i error

# Warning messages
tail -100 /var/log/grafana/grafana.log | grep -i warn
docker logs grafana --tail 100 2>&1 | grep -i warn

# Database connection errors
grep "database.*error\|db.*error\|connection.*failed" /var/log/grafana/grafana.log | tail -20

# Plugin errors
grep "plugin.*error\|plugin.*failed" /var/log/grafana/grafana.log | tail -20

# Query timeout errors
grep "timeout\|timed out" /var/log/grafana/grafana.log | tail -20

# Follow live logs (native)
tail -f /var/log/grafana/grafana.log

# Follow live logs (Docker)
docker logs -f grafana

# Log rotation status
ls -lh /var/log/grafana/
cat /etc/logrotate.d/grafana 2>/dev/null

# Critical errors only
grep -E "error|critical|fatal" /var/log/grafana/grafana.log | tail -50
```

## Docker-Specific Diagnostics
```bash
# Container status
docker inspect grafana | jq

# Container state
docker inspect grafana | jq '.[0].State'

# Resource limits
docker inspect grafana | jq '.[0].HostConfig | {Memory, MemorySwap, CpuShares}'

# Container stats
docker stats grafana --no-stream

# Volume mounts
docker inspect grafana | jq '.[0].Mounts'

# Network settings
docker inspect grafana | jq '.[0].NetworkSettings'

# Environment variables
docker exec grafana env | grep GF_
docker inspect grafana | jq '.[0].Config.Env'

# Container restart policy
docker inspect grafana | jq '.[0].HostConfig.RestartPolicy'

# Container logs size
docker inspect grafana | jq '.[0].LogPath'
du -h $(docker inspect grafana | jq -r '.[0].LogPath')

# Port mappings
docker port grafana

# Container health check (if configured)
docker inspect grafana | jq '.[0].State.Health'
```

## Common Issues & Troubleshooting
```bash
# Check for "database locked" errors
grep "database.*locked" /var/log/grafana/grafana.log | tail -20

# File permissions check
ls -la /var/lib/grafana/
ls -la /etc/grafana/
ls -la /var/log/grafana/

# Ownership check
stat /var/lib/grafana/ | grep Uid
stat /var/lib/grafana/grafana.db | grep Uid

# Test API connectivity
curl -v http://localhost:3000/api/health 2>&1 | grep -E "Connected|HTTP"

# Check for port conflicts
lsof -i :3000
ss -tlnp | grep :3000

# Memory issues
free -h
ps aux | grep grafana | awk '{print $4, $11}'

# Database migration issues
grep "migration\|migrate" /var/log/grafana/grafana.log | tail -20

# Plugin compatibility
grep "plugin.*incompatible\|plugin.*deprecated" /var/log/grafana/grafana.log

# Configuration syntax errors
grep "config.*error\|invalid.*config" /var/log/grafana/grafana.log | tail -20

# Disk space
df -h /var/lib/grafana
df -h /var/log/grafana
```

## Output Format
Provide a Grafana diagnostic report with:
- Service Status
  - Running/Stopped
  - Version
  - Uptime
  - Installation type (native/Docker)
- Datasource Health
  - Total datasources configured
  - Connected datasources (with names)
  - Failed datasources (with error messages)
- Dashboard Status
  - Total dashboards
  - Recently modified dashboards
  - Dashboard errors from logs
- Alert Status
  - Active alert rules
  - Firing alerts
  - Notification channel status
  - Failed notifications
- Performance Metrics
  - CPU usage %
  - Memory usage (MB)
  - Average query response time
  - Slow queries detected
- Database Backend
  - Database type (SQLite/PostgreSQL/MySQL)
  - Database size
  - Connection status
  - Database errors
- Authentication
  - Total users
  - Active sessions
  - Authentication method (local/LDAP/OAuth)
  - Recent auth failures
- Issues Found
  - Critical errors from logs
  - Failed datasources
  - Performance bottlenecks
  - Configuration problems
  - Permission issues
- Recommendations
  - Immediate actions needed
  - Performance tuning suggestions
  - Security improvements
  - Upgrade considerations

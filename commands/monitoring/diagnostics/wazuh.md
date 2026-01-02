# /monitoring-diagnostics-wazuh

Diagnose Wazuh manager and agent status, alerts, rules, log processing, and cluster health.

## Usage
`/monitoring-diagnostics-wazuh`
`/monitoring-diagnostics-wazuh --manager`
`/monitoring-diagnostics-wazuh --agents`
`/monitoring-diagnostics-wazuh --cluster`

## Tasks
1. Check Wazuh manager service status
2. Verify agent connections and health
3. Review cluster synchronization status
4. Analyze recent alerts by severity
5. Check rule hits and compliance status
6. Monitor log processing and event rates
7. Verify queue status and ingestion health
8. Generate comprehensive diagnostic report

## Service Detection & Status
```bash
# Wazuh manager status (all daemons)
/var/ossec/bin/wazuh-control status

# Individual daemon checks
pgrep -a wazuh-remoted    # Agent communication
pgrep -a wazuh-analysisd  # Event analysis
pgrep -a wazuh-logcollector  # Log collection
pgrep -a wazuh-monitord   # Internal monitoring
pgrep -a wazuh-modulesd   # Modules daemon
pgrep -a wazuh-db         # Database daemon
pgrep -a wazuh-authd      # Agent enrollment
pgrep -a wazuh-clusterd   # Cluster management

# Systemd service status
systemctl status wazuh-manager

# Wazuh version
/var/ossec/bin/wazuh-control info

# Process resource usage
ps aux | grep wazuh- | grep -v grep

# Daemon uptime
ps -eo pid,etime,cmd | grep wazuh- | grep -v grep

# Service enabled on boot
systemctl is-enabled wazuh-manager 2>/dev/null
```

## Manager Health Check
```bash
# API health endpoint
curl -k -X GET "https://localhost:55000/" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# Manager status via API
curl -k -X GET "https://localhost:55000/manager/status" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# Manager info
curl -k -X GET "https://localhost:55000/manager/info" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# Check critical logs
tail -100 /var/ossec/logs/ossec.log | grep -E "ERROR|CRITICAL"

# Database connectivity
pgrep wazuh-db && echo "Wazuh DB running" || echo "Wazuh DB not running"

# API logs
tail -50 /var/ossec/logs/api.log 2>/dev/null

# Service crashes
journalctl -u wazuh-manager | grep -E "failed|crashed|core dump" | tail -20

# Manager configuration test
/var/ossec/bin/wazuh-logtest < /dev/null 2>&1 | head -5

# Queue status from manager
/var/ossec/bin/wazuh-control status | grep -i queue
```

## Agent Status & Connectivity
```bash
# List all agents
/var/ossec/bin/agent_control -l

# List active/connected agents
/var/ossec/bin/agent_control -lc

# List disconnected agents
/var/ossec/bin/agent_control -ln

# Never connected agents
/var/ossec/bin/agent_control -l | grep "Never connected"

# Specific agent info
/var/ossec/bin/agent_control -i 001

# Agent summary
echo "Total: $(/var/ossec/bin/agent_control -l | grep -c 'ID:')"
echo "Active: $(/var/ossec/bin/agent_control -lc | grep -c 'ID:')"
echo "Disconnected: $(/var/ossec/bin/agent_control -ln | grep -c 'ID:')"

# API: List all agents
curl -k -X GET "https://localhost:55000/agents" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# API: Active agents only
curl -k -X GET "https://localhost:55000/agents?status=active" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# Agent connection logs
grep "Agent.*connected" /var/ossec/logs/ossec.log | tail -50

# Agent disconnection events
grep "Agent.*disconnected\|Agent.*removed" /var/ossec/logs/ossec.log | tail -30

# Last keep-alive timestamps
/var/ossec/bin/agent_control -l | grep -A 1 "ID:"

# Agent version distribution
curl -k -X GET "https://localhost:55000/agents/summary/os" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# Outdated agents
curl -k -X GET "https://localhost:55000/agents/outdated" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# Agent enrollment issues
grep "auth.*error\|Unable to add agent" /var/ossec/logs/ossec.log | tail -20
```

## Cluster Health & Synchronization
```bash
# Cluster status
/var/ossec/bin/cluster_control -l

# Cluster health
/var/ossec/bin/cluster_control -a

# Node information
/var/ossec/bin/cluster_control -i

# Connected nodes
/var/ossec/bin/cluster_control -n

# Cluster configuration
grep -A 20 "<cluster>" /var/ossec/etc/ossec.conf

# API: Cluster status
curl -k -X GET "https://localhost:55000/cluster/status" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# API: Cluster health
curl -k -X GET "https://localhost:55000/cluster/healthcheck" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# API: Cluster nodes
curl -k -X GET "https://localhost:55000/cluster/nodes" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# Synchronization status
grep "Integrity check" /var/ossec/logs/cluster.log | tail -10
grep "Agent-info sync" /var/ossec/logs/cluster.log | tail -10
grep "Agent-groups sync" /var/ossec/logs/cluster.log | tail -10

# Master node check
/var/ossec/bin/cluster_control -i | grep "Type"

# Cluster communication errors
grep "Error.*cluster\|cluster.*connection" /var/ossec/logs/cluster.log | tail -20

# Cluster synchronization issues
grep "sync.*failed\|sync.*error" /var/ossec/logs/cluster.log | tail -20

# Cluster member count
/var/ossec/bin/cluster_control -l | grep -c "node"
```

## Alert Analysis
```bash
# Recent alerts (JSON format)
tail -100 /var/ossec/logs/alerts/alerts.json

# Parse alerts with jq
cat /var/ossec/logs/alerts/alerts.json | tail -20 | jq

# Alerts by severity (level 10+)
cat /var/ossec/logs/alerts/alerts.json | jq 'select(.rule.level >= 10)' | tail -20

# Critical alerts (level 12+)
cat /var/ossec/logs/alerts/alerts.json | jq 'select(.rule.level >= 12)' | tail -10

# Count alerts by level
jq -r '.rule.level' /var/ossec/logs/alerts/alerts.json | sort | uniq -c | sort -rn

# Alerts by rule ID
jq -r '.rule.id' /var/ossec/logs/alerts/alerts.json | sort | uniq -c | sort -rn | head -20

# Top alert sources
jq -r '.agent.name // "manager"' /var/ossec/logs/alerts/alerts.json | sort | uniq -c | sort -rn | head -15

# Alerts in last hour
find /var/ossec/logs/alerts -name "*.json" -mmin -60 -exec cat {} \; | jq -r '[.timestamp, .rule.level, .rule.description] | @tsv' | tail -30

# Alert statistics
echo "Total alerts: $(cat /var/ossec/logs/alerts/alerts.json 2>/dev/null | wc -l)"
echo "High severity (>10): $(jq 'select(.rule.level > 10)' /var/ossec/logs/alerts/alerts.json 2>/dev/null | wc -l)"

# Specific rule alerts
jq "select(.rule.id == \"5710\")" /var/ossec/logs/alerts/alerts.json | tail -10

# Failed login attempts
jq 'select(.rule.groups[]? == "authentication_failed")' /var/ossec/logs/alerts/alerts.json | tail -20
```

## Rule Status & Compliance
```bash
# List custom rules
ls -lh /var/ossec/etc/rules/local_rules.xml
cat /var/ossec/etc/rules/local_rules.xml

# All rule files
ls -lh /var/ossec/ruleset/rules/

# Check rule syntax (requires logtest)
echo "test" | /var/ossec/bin/wazuh-logtest 2>&1 | head -10

# Decoder files
ls -lh /var/ossec/etc/decoders/
cat /var/ossec/etc/decoders/local_decoder.xml 2>/dev/null

# CDB lists
ls -lh /var/ossec/etc/lists/

# Rule compilation errors
grep "rule.*error\|invalid rule" /var/ossec/logs/ossec.log | tail -20

# Compliance framework alerts (PCI-DSS)
jq 'select(.rule.pci_dss != null)' /var/ossec/logs/alerts/alerts.json | tail -10

# GDPR compliance alerts
jq 'select(.rule.gdpr != null)' /var/ossec/logs/alerts/alerts.json | tail -10

# HIPAA compliance alerts
jq 'select(.rule.hipaa != null)' /var/ossec/logs/alerts/alerts.json | tail -10

# Rule statistics from API
curl -k -X GET "https://localhost:55000/rules" -H "Authorization: Bearer $WAZUH_TOKEN" | jq '.data.total_affected_items'

# Disabled rules
grep "disabled" /var/ossec/etc/ossec.conf | grep rule

# Rule update status
ls -lt /var/ossec/ruleset/rules/ | head -10
```

## Log Processing & Event Rates
```bash
# Event processing statistics
grep "Total events" /var/ossec/logs/ossec.log | tail -20

# Events per second (EPS)
grep "eps" /var/ossec/logs/ossec.log | tail -10

# Log collector status
pgrep wazuh-logcollector && /var/ossec/bin/agent_control -m | head -20

# Archives (all events)
tail -100 /var/ossec/logs/archives/archives.json

# Syslog collection
grep "syslog" /var/ossec/etc/ossec.conf | grep -v "<!--"

# File monitoring (FIM) events
jq 'select(.syscheck != null)' /var/ossec/logs/alerts/alerts.json | tail -20

# Dropped events
grep "dropped\|Dropping" /var/ossec/logs/ossec.log | tail -20

# Processing delays
grep "delay\|backlog" /var/ossec/logs/ossec.log | tail -20

# Analysis daemon performance
ps aux | grep wazuh-analysisd
grep "analysisd" /var/ossec/logs/ossec.log | tail -30

# Events by category
jq -r '.rule.groups[]?' /var/ossec/logs/alerts/alerts.json | sort | uniq -c | sort -rn | head -20

# Log sources
grep "ossec-logcollector" /var/ossec/logs/ossec.log | grep "Analyzing file" | tail -20
```

## Queue Status & Ingestion
```bash
# Queue overview
ls -lh /var/ossec/queue/

# Remote daemon queue
ss -s | grep -A 5 "TCP:"

# Agent communication port
ss -tlnp | grep 1514
netstat -tlnp | grep 1514

# Analysis queue
ls -lh /var/ossec/queue/diff/
ls -lh /var/ossec/queue/rids/

# Event queue size
find /var/ossec/queue -type f | wc -l

# Queue overflow warnings
grep "queue.*full\|queue.*overflow\|queue.*flooded" /var/ossec/logs/ossec.log | tail -20

# Message backlog
grep "backlog\|messages pending" /var/ossec/logs/ossec.log | tail -10

# TCP vs UDP agent connections
ss -tuln | grep 1514

# Agent buffer status
grep "agent buffer" /var/ossec/logs/ossec.log | tail -10

# Remoted daemon statistics
grep "wazuh-remoted" /var/ossec/logs/ossec.log | grep "Total" | tail -10

# Database queue
du -sh /var/ossec/queue/db/
ls -lh /var/ossec/queue/db/ | wc -l
```

## Database Status
```bash
# Wazuh DB process
pgrep -a wazuh-db

# Database size
du -sh /var/ossec/queue/db/

# Individual agent databases
ls -lh /var/ossec/queue/db/*.db | head -20

# Database integrity
sqlite3 /var/ossec/queue/db/global.db "PRAGMA integrity_check;" 2>/dev/null

# Database errors
grep "wazuh-db.*error\|database.*error" /var/ossec/logs/ossec.log | tail -30

# Database query performance
grep "wazuh-db.*query" /var/ossec/logs/ossec.log | tail -20

# Database synchronization
grep "wazuh-db.*sync" /var/ossec/logs/ossec.log | tail -10

# Agent database files
find /var/ossec/queue/db -name "*.db" | wc -l

# Database vacuum status
grep "vacuum" /var/ossec/logs/ossec.log | tail -10

# Global database tables
sqlite3 /var/ossec/queue/db/global.db ".tables" 2>/dev/null
```

## API Diagnostics
```bash
# API authentication (get token)
curl -u wazuh:wazuh -k -X GET "https://localhost:55000/security/user/authenticate" | jq

# API status
curl -k -X GET "https://localhost:55000/" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# API configuration
grep -A 30 "<api>" /var/ossec/etc/ossec.conf

# API logs
tail -100 /var/ossec/logs/api.log

# API errors
grep -E "error|ERROR|fail|FAIL" /var/ossec/logs/api.log | tail -30

# API access logs
grep "200\|404\|500" /var/ossec/logs/api.log | tail -30

# API authentication failures
grep "authentication.*fail\|Unauthorized" /var/ossec/logs/api.log | tail -20

# API endpoints test
curl -k -X GET "https://localhost:55000/agents/summary/status" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# API RBAC permissions
curl -k -X GET "https://localhost:55000/security/roles" -H "Authorization: Bearer $WAZUH_TOKEN" | jq

# API rate limiting
grep "rate limit" /var/ossec/logs/api.log | tail -10

# API listening port
ss -tlnp | grep 55000
```

## File Integrity Monitoring
```bash
# FIM database location
ls -lh /var/ossec/queue/fim/db/

# Recent FIM alerts
jq 'select(.syscheck != null)' /var/ossec/logs/alerts/alerts.json | tail -30

# FIM configuration
grep -A 20 "<syscheck>" /var/ossec/etc/ossec.conf

# FIM scan status
grep "syscheck scan" /var/ossec/logs/ossec.log | tail -20

# Monitored directories
grep "<directories" /var/ossec/etc/ossec.conf | grep -v "<!--"

# FIM event frequency
jq 'select(.syscheck != null)' /var/ossec/logs/alerts/alerts.json | jq -r '.syscheck.event' | sort | uniq -c

# Real-time monitoring
grep "realtime" /var/ossec/etc/ossec.conf

# FIM database size
du -sh /var/ossec/queue/fim/db/

# File changes detected
jq 'select(.syscheck.event == "modified")' /var/ossec/logs/alerts/alerts.json | tail -10
```

## Log Files & Troubleshooting
```bash
# Main ossec log
tail -100 /var/ossec/logs/ossec.log

# Cluster log
tail -100 /var/ossec/logs/cluster.log

# API log
tail -100 /var/ossec/logs/api.log

# Integration logs (if configured)
ls -lh /var/ossec/logs/integrations/

# Follow main log
tail -f /var/ossec/logs/ossec.log

# All errors
grep -i "error" /var/ossec/logs/ossec.log | tail -50

# All warnings
grep -i "warn" /var/ossec/logs/ossec.log | tail -30

# Configuration errors
grep "config.*error\|invalid.*config" /var/ossec/logs/ossec.log | tail -20

# Permission errors
grep "permission denied\|cannot open" /var/ossec/logs/ossec.log | tail -20

# Authentication errors
grep "authentication.*error\|auth.*fail" /var/ossec/logs/ossec.log | tail -20

# Log rotation
ls -lh /var/ossec/logs/*.log
cat /etc/logrotate.d/wazuh 2>/dev/null

# Integration errors (Slack, email, etc.)
grep "integration.*error\|slack.*error" /var/ossec/logs/integrations/ 2>/dev/null | tail -20
```

## Common Issues & Fixes
```bash
# Disk space check
df -h /var/ossec

# Wazuh directory permissions
ls -la /var/ossec/ | head -20

# File ownership
stat /var/ossec | grep Uid

# Port availability
ss -tlnp | grep -E "1514|1515|55000"

# Firewall rules (iptables)
iptables -L -n | grep -E "1514|1515|55000"

# Firewall rules (firewalld)
firewall-cmd --list-all 2>/dev/null | grep -E "1514|1515|55000"

# SELinux context (RHEL/CentOS)
getenforce 2>/dev/null
ls -lZ /var/ossec | head -10

# Time synchronization
timedatectl status
ntpq -p 2>/dev/null || chronyc sources 2>/dev/null

# Certificate issues
openssl s_client -connect localhost:55000 </dev/null 2>&1 | grep -E "Verify|Certificate"

# Agent enrollment issues
grep "Unable to add agent\|agent.*enrollment" /var/ossec/logs/ossec.log | tail -20

# Configuration syntax check
/var/ossec/bin/wazuh-control status | grep -i "configuration"

# Memory usage
free -h
ps aux | grep wazuh | awk '{sum+=$6} END {print "Wazuh memory: " sum/1024 " MB"}'

# Open file descriptors
lsof -u ossec 2>/dev/null | wc -l
cat /proc/$(pgrep wazuh-remoted)/limits | grep "open files"
```

## Output Format
Provide a Wazuh diagnostic report with:
- Manager Status
  - Running/Stopped
  - Wazuh version
  - All daemon status
  - Manager uptime
- Agent Statistics
  - Total agents
  - Active agents (count + percentage)
  - Disconnected agents (count + names)
  - Never connected agents
  - Outdated agents
- Cluster Health (if configured)
  - Cluster enabled/disabled
  - Node type (master/worker)
  - Connected nodes (count + names)
  - Synchronization status
  - Cluster errors
- Alert Summary
  - Total alerts (last hour/day)
  - Alerts by severity level
  - Top 10 triggered rules
  - Top alert sources
  - Compliance alerts (PCI-DSS, GDPR, HIPAA)
- Performance Metrics
  - Events per second (EPS)
  - Queue sizes
  - Processing delays
  - Memory usage
  - Database size
- Rule & Compliance Status
  - Total active rules
  - Custom rules count
  - Compliance frameworks enabled
  - Rule errors
- Log Processing
  - Event processing rate
  - FIM events
  - Dropped events
  - Log sources monitored
- Issues Found
  - Critical errors from logs
  - Disconnected agents
  - Queue overflows
  - Synchronization failures
  - Configuration errors
  - Permission/security issues
- Recommendations
  - Agents to investigate
  - Capacity planning suggestions
  - Security hardening steps
  - Performance tuning recommendations
  - Configuration improvements

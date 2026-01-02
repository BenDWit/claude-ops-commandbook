# /linux-diagnostics-performance

Diagnose system performance issues including CPU, memory, disk I/O, and resource bottlenecks.

## Usage
`/linux-diagnostics-performance`
`/linux-diagnostics-performance --cpu`
`/linux-diagnostics-performance --memory`
`/linux-diagnostics-performance --disk`
`/linux-diagnostics-performance --full`

## Tasks
1. Check overall system load and uptime
2. Analyze CPU usage and top processes
3. Check memory and swap usage
4. Analyze disk I/O and performance
5. Check for resource bottlenecks
6. Review system limits and quotas
7. Identify performance issues
8. Generate performance report

## System Overview
```bash
# System uptime and load averages
uptime
w

# Load average explanation:
# 1-min, 5-min, 15-min averages
# Compare to number of CPU cores
nproc

# Quick system stats
top -bn1 | head -20
htop -C  # if installed

# Overall resource usage
vmstat 1 5
mpstat 1 5  # if sysstat installed
```

## CPU Diagnostics
```bash
# CPU information
lscpu
cat /proc/cpuinfo | grep "model name" | uniq

# CPU cores and threads
nproc
grep -c processor /proc/cpuinfo

# Current CPU usage
top -bn1 | grep "Cpu(s)"
mpstat

# Per-core CPU usage
mpstat -P ALL 1 5

# Top CPU consuming processes
ps aux --sort=-%cpu | head -20

# CPU usage over time (if sar installed)
sar -u 1 10

# Check for CPU throttling
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq 2>/dev/null
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor 2>/dev/null

# CPU steal time (important for VMs)
top -bn1 | grep "%Cpu"
# High 'st' (steal) indicates hypervisor resource contention
```

## Memory Diagnostics
```bash
# Memory overview
free -h
cat /proc/meminfo

# Detailed memory usage
vmstat -s
vmstat 1 5

# Top memory consuming processes
ps aux --sort=-%mem | head -20

# Memory by process tree
ps auxf | head -50

# Check for OOM (Out of Memory) kills
dmesg | grep -i "killed process"
grep -i "out of memory" /var/log/syslog
journalctl -k | grep -i "killed process"

# Cache and buffer usage
sync; echo 3 > /proc/sys/vm/drop_caches  # Clear caches (careful!)
# Don't run the above unless necessary

# Swap usage
swapon --show
cat /proc/swaps
vmstat | grep swap

# Processes using swap
for file in /proc/*/status ; do
  awk '/VmSwap|Name/{printf $2 " " $3}END{ print ""}' $file
done | sort -k 2 -n -r | head -20

# Memory allocation per user
ps aux | awk '{mem[$1]+=$6} END {for (user in mem) print user, mem[user]/1024 "MB"}' | sort -k2 -rn
```

## Disk I/O Diagnostics
```bash
# Disk usage
df -h
df -i  # Inode usage

# I/O statistics (requires sysstat)
iostat -x 1 5
iostat -d -x 1 5

# Per-process I/O
iotop -b -n 3  # if installed
pidstat -d 1 5

# Disk I/O wait
vmstat 1 5
# High 'wa' (I/O wait) indicates disk bottleneck

# Check for disk errors
dmesg | grep -i error
smartctl -a /dev/sda  # if smartmontools installed

# Inode usage by directory
for i in /*; do
  echo "$i: $(find $i 2>/dev/null | wc -l) files"
done | sort -k2 -rn

# Large files
du -ah / 2>/dev/null | sort -rh | head -30

# Find what's using disk space
du -sh /* 2>/dev/null | sort -rh
ncdu /  # if installed (interactive)
```

## Process Analysis
```bash
# All processes sorted by resource usage
ps auxf --sort=-%mem | head -30
ps auxf --sort=-%cpu | head -30

# Process count
ps aux | wc -l

# Process states
ps aux | awk '{print $8}' | sort | uniq -c

# Zombie processes
ps aux | grep Z
ps aux | awk '$8=="Z" {print}'

# Process limits
cat /proc/sys/kernel/pid_max
ulimit -a

# Threads per process
ps -eLf | wc -l
ps -eo nlwp,pid,cmd --sort=-nlwp | head -20
```

## System Limits and Bottlenecks
```bash
# File descriptor usage
lsof | wc -l
cat /proc/sys/fs/file-nr

# Max file descriptors
cat /proc/sys/fs/file-max
ulimit -n

# Per-process file descriptors
lsof -p <PID> | wc -l
ls -l /proc/<PID>/fd | wc -l

# Open files by process
lsof | awk '{print $1}' | sort | uniq -c | sort -rn | head -20

# Network connections count
ss -s
netstat -an | wc -l

# Context switches (high = CPU contention)
vmstat 1 5
# 'cs' column shows context switches

# Interrupts
cat /proc/interrupts
```

## Network Performance
```bash
# Network interface statistics
ifconfig -a
ip -s link

# Network throughput
sar -n DEV 1 5  # if sysstat installed
ifstat 1 5      # if installed

# Bandwidth usage
iftop  # if installed
nethogs  # if installed

# Connection states
ss -s
netstat -an | awk '{print $6}' | sort | uniq -c

# TCP performance settings
sysctl net.ipv4.tcp_congestion_control
sysctl net.core.rmem_max
sysctl net.core.wmem_max
```

## Load Testing Tools
```bash
# Generate CPU load (for testing)
# stress --cpu 4 --timeout 60s  # if installed

# Monitor system during load
watch -n 1 'uptime; free -h'

# Record performance data
sar -A -o /tmp/perf.log 60 10  # Record for 10 minutes
# View later with: sar -f /tmp/perf.log
```

## Historical Performance Data
```bash
# SAR historical data (if sysstat configured)
sar -u  # CPU usage yesterday
sar -r  # Memory usage yesterday
sar -d  # Disk I/O yesterday
sar -n DEV  # Network yesterday

# List available SAR logs
ls -lh /var/log/sysstat/
ls -lh /var/log/sa/

# Specific date
sar -f /var/log/sysstat/sa$(date +%d -d yesterday)
```

## Kernel Parameters
```bash
# View all kernel parameters
sysctl -a

# Important performance parameters
sysctl vm.swappiness
sysctl vm.dirty_ratio
sysctl vm.dirty_background_ratio
sysctl net.core.somaxconn
sysctl fs.file-max

# Check for tuning
cat /etc/sysctl.conf
cat /etc/sysctl.d/*.conf
```

## Database Performance (if applicable)
```bash
# MySQL connections
mysqladmin processlist 2>/dev/null
mysqladmin status 2>/dev/null

# PostgreSQL connections
ps aux | grep postgres

# Check database processes
ps aux | grep -E 'mysql|postgres|mongo' | grep -v grep
```

## Common Performance Issues
```bash
# Check for runaway processes
ps aux --sort=-%cpu | head -5
ps aux --sort=-%mem | head -5

# Check system logs for errors
journalctl -p err -n 50
tail -100 /var/log/syslog | grep -i error

# Check for hardware issues
dmesg | grep -i "hardware error"
mcelog --client  # if installed

# Temperature monitoring (if sensors installed)
sensors

# Check for resource limits
ulimit -a
cat /etc/security/limits.conf
cat /etc/security/limits.d/*.conf
```

## Performance Monitoring Setup
```bash
# Install monitoring tools
# Debian/Ubuntu:
apt install sysstat iotop htop iftop nethogs dstat

# RHEL/CentOS:
yum install sysstat iotop htop iftop nethogs dstat

# Enable sysstat collection
systemctl enable sysstat
systemctl start sysstat
# Edit /etc/default/sysstat and set ENABLED="true"
```

## Quick Performance Tuning
```bash
# Reduce swappiness (prefer RAM over swap)
sysctl vm.swappiness=10
echo "vm.swappiness=10" >> /etc/sysctl.conf

# Increase file descriptor limit
ulimit -n 65535
# Add to /etc/security/limits.conf:
# * soft nofile 65535
# * hard nofile 65535

# TCP tuning for high-performance servers
cat >> /etc/sysctl.conf <<EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
EOF
sysctl -p
```

## Output Format
Provide a performance diagnostic report with:
- System Overview
  - Uptime and load averages
  - CPU count vs load
- CPU Status
  - Current usage %
  - Top CPU processes
  - Load trends
- Memory Status
  - Total/Used/Available
  - Swap usage
  - Top memory processes
  - OOM events
- Disk I/O Status
  - Disk usage %
  - I/O wait time
  - Top I/O processes
- Bottlenecks Identified
  - CPU-bound
  - Memory-bound
  - I/O-bound
  - Network-bound
- Recommendations
  - Immediate actions
  - Tuning suggestions
  - Resource allocation changes

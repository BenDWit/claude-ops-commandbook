# /linux-diagnostics-process

Diagnose process-related issues including hung processes, zombies, resource hogs, and service problems.

## Usage
`/linux-diagnostics-process`
`/linux-diagnostics-process --zombies`
`/linux-diagnostics-process --high-cpu`
`/linux-diagnostics-process --high-memory`
`/linux-diagnostics-process --service <name>`

## Tasks
1. List all running processes
2. Identify resource-intensive processes
3. Find zombie and defunct processes
4. Check process limits and quotas
5. Analyze process relationships (parent/child)
6. Check for hung or unresponsive processes
7. Review service status
8. Generate process diagnostic report

## Process Overview
```bash
# All processes
ps aux
ps auxf  # Tree view

# Top processes by CPU
ps aux --sort=-%cpu | head -20

# Top processes by memory
ps aux --sort=-%mem | head -20

# Process tree
pstree -p
ps auxf

# Process count
ps aux | wc -l

# Interactive process monitor
top
htop  # if installed
```

## CPU-Heavy Processes
```bash
# Top CPU consumers
ps aux --sort=-%cpu | head -20

# Processes using >50% CPU
ps aux | awk '$3 > 50 {print}'

# Monitor specific process CPU
top -p <PID>
pidstat -p <PID> 1

# Per-thread CPU usage
top -H -p <PID>
ps -eLo pid,tid,class,rtprio,ni,pri,psr,pcpu,stat,wchan:14,comm

# CPU usage history
sar -u 1 10  # if sysstat installed

# Process nice values
ps axo pid,ni,comm --sort=-ni
```

## Memory-Heavy Processes
```bash
# Top memory consumers
ps aux --sort=-%mem | head -20

# Processes using >1GB memory
ps aux | awk '$6 > 1000000 {print $0}'

# Virtual vs Resident memory
ps axo pid,comm,vsz,rss,pmem --sort=-rss | head -20

# Memory map of process
pmap <PID>
pmap -x <PID>

# Detailed memory info
cat /proc/<PID>/status | grep -E "Vm|Rss"
cat /proc/<PID>/smaps_rollup

# Total memory by user
ps aux | awk '{mem[$1]+=$6} END {for (user in mem) print user, mem[user]/1024 "MB"}' | sort -k2 -rn

# Process memory over time
pidstat -r -p <PID> 1
```

## Zombie Processes
```bash
# Find zombie processes
ps aux | grep Z
ps aux | awk '$8=="Z" {print}'

# Count zombies
ps aux | grep -c Z

# Zombie with parent info
ps -eo pid,ppid,stat,comm | grep Z

# Parent of zombie
ps -o pid,ppid,comm,stat -p <ZOMBIE_PID>
ps -p <PARENT_PID> -o pid,cmd

# Kill parent to clean zombies (carefully!)
# kill -9 <PARENT_PID>
```

## Defunct and Stuck Processes
```bash
# Defunct processes
ps aux | grep defunct

# Processes in uninterruptible sleep (D state)
ps aux | awk '$8=="D" {print}'
ps -eo pid,stat,cmd | grep "^[0-9]* D"

# Stuck processes waiting on I/O
ps -eo pid,stat,wchan,comm | grep "^[0-9]* D"

# Process state breakdown
ps aux | awk '{print $8}' | sort | uniq -c | sort -rn
```

## Process Relationships
```bash
# Process tree
pstree -p
pstree -u  # With usernames
pstree -a  # With arguments

# Children of specific process
pgrep -P <PARENT_PID>
ps --ppid <PARENT_PID>

# Process hierarchy
ps axjf

# All processes by user
ps -u <username>
pgrep -u <username>

# Process with command arguments
ps aux | grep <pattern>
pgrep -af <pattern>
```

## Process Details
```bash
# Full process info
ps -p <PID> -f
ps -p <PID> -o pid,ppid,cmd,stat,time,%cpu,%mem

# Process command line
cat /proc/<PID>/cmdline | tr '\0' ' '

# Process environment
cat /proc/<PID>/environ | tr '\0' '\n'

# Process open files
lsof -p <PID>

# Process network connections
lsof -i -a -p <PID>
ss -p | grep "pid=<PID>"

# Process working directory
pwdx <PID>
ls -l /proc/<PID>/cwd

# Process file descriptors
ls -l /proc/<PID>/fd

# Process limits
cat /proc/<PID>/limits

# Process threads
ps -eLf | grep <PID>
cat /proc/<PID>/status | grep Threads
```

## Thread Analysis
```bash
# Threads per process
ps -eLf | wc -l

# Processes sorted by thread count
ps -eo nlwp,pid,cmd --sort=-nlwp | head -20

# Threads of specific process
ps -eLf -p <PID>
top -H -p <PID>

# Thread CPU usage
pidstat -t -p <PID> 1
```

## Process Limits
```bash
# System-wide limits
cat /proc/sys/kernel/pid_max
cat /proc/sys/kernel/threads-max

# Current process count
ps aux | wc -l

# User limits
ulimit -a

# Limits for specific process
cat /proc/<PID>/limits

# File descriptor limits
cat /proc/sys/fs/file-max
cat /proc/sys/fs/file-nr

# Per-user process limits
cat /etc/security/limits.conf
cat /etc/security/limits.d/*.conf
```

## Service Management
```bash
# All services status
systemctl list-units --type=service

# Failed services
systemctl --failed --type=service

# Running services
systemctl list-units --type=service --state=running

# Specific service status
systemctl status <service>

# Service dependencies
systemctl list-dependencies <service>

# Service resource usage
systemd-cgtop
systemctl status <service> | grep -E "Memory|CPU"

# Service logs
journalctl -u <service> -n 100
journalctl -u <service> -f  # Follow
```

## Signals and Process Control
```bash
# Send signal to process
kill -<SIGNAL> <PID>

# Common signals
kill -15 <PID>  # SIGTERM (graceful)
kill -9 <PID>   # SIGKILL (force)
kill -1 <PID>   # SIGHUP (reload config)

# Kill all processes matching name
pkill <process_name>
killall <process_name>

# Kill all processes by user
pkill -u <username>

# Check if process responds to signals
kill -0 <PID>  # Check if alive
```

## Process Priorities
```bash
# Show nice values
ps axo pid,ni,comm --sort=-ni

# Change priority (renice)
renice -n 10 -p <PID>
renice -n -5 -p <PID>  # Needs root

# Real-time priority
ps -eo pid,rtprio,comm

# Start process with priority
nice -n 10 <command>
```

## Background and Stopped Processes
```bash
# List jobs in current shell
jobs

# Background running processes
jobs -r

# Stopped processes
jobs -s

# Bring to foreground
fg %<job_number>

# Send to background
bg %<job_number>

# Disown process (keep running after logout)
disown -h %<job_number>
```

## Process Monitoring
```bash
# Watch process in real-time
watch -n 1 'ps aux | grep <pattern>'

# Monitor specific PID
pidstat -p <PID> 1

# CPU usage
pidstat -u -p <PID> 1

# Memory usage
pidstat -r -p <PID> 1

# I/O usage
pidstat -d -p <PID> 1

# Context switches
pidstat -w -p <PID> 1

# All stats
pidstat -u -r -d -p <PID> 1
```

## Cron and Scheduled Jobs
```bash
# User crontabs
crontab -l
crontab -l -u <username>

# System cron jobs
cat /etc/crontab
ls -la /etc/cron.*

# All user cron jobs
for user in $(cut -f1 -d: /etc/passwd); do
  echo "=== $user ==="
  crontab -l -u $user 2>/dev/null
done

# Systemd timers
systemctl list-timers --all

# At jobs
atq
at -l
```

## Process Debugging
```bash
# Trace system calls
strace -p <PID>
strace -c -p <PID>  # Summary

# Trace library calls
ltrace -p <PID>

# Process stack trace
gdb -p <PID> -batch -ex "thread apply all bt"

# perf profiling (if installed)
perf top
perf record -p <PID> -g -- sleep 10
perf report

# Generate core dump
gcore <PID>

# Process hang detection
cat /proc/<PID>/stack
```

## Resource Cgroups
```bash
# List cgroups
systemd-cgls

# Resource usage by cgroup
systemd-cgtop

# Cgroup limits
cat /sys/fs/cgroup/*/memory.limit_in_bytes 2>/dev/null
cat /sys/fs/cgroup/*/cpu.cfs_quota_us 2>/dev/null
```

## Common Process Issues
```bash
# Find processes running long time
ps -eo pid,etime,cmd --sort=-etime | head -20

# Processes with high page faults
ps -eo pid,maj_flt,min_flt,cmd --sort=-maj_flt | head -20

# Find processes in specific state
ps -eo pid,stat,cmd | grep "R"  # Running
ps -eo pid,stat,cmd | grep "S"  # Sleeping
ps -eo pid,stat,cmd | grep "D"  # Uninterruptible sleep
ps -eo pid,stat,cmd | grep "T"  # Stopped
ps -eo pid,stat,cmd | grep "Z"  # Zombie

# Process swap usage
for pid in $(pgrep -x <process_name>); do
  echo "PID: $pid"
  grep VmSwap /proc/$pid/status
done
```

## Quick Fixes
```bash
# Restart hung service
systemctl restart <service>

# Force kill process
kill -9 <PID>

# Kill all instances of program
killall -9 <program_name>

# Clear defunct processes (kill parent)
ps -A -ostat,ppid | grep -e '[zZ]' | awk '{print $2}' | uniq | xargs kill -9

# Reload service config
systemctl reload <service>

# Restart after crash
systemctl enable <service>  # Auto-restart on boot
# Or edit service file and add:
# Restart=always
# RestartSec=10s
```

## Process Accounting
```bash
# Enable process accounting (if not enabled)
accton /var/log/account/pacct

# Process accounting stats
sa  # if psacct installed
lastcomm  # Recently executed commands

# Top commands run
sa -u
```

## Output Format
Provide a process diagnostic report with:
- Process Overview
  - Total process count
  - Processes by state
  - Top resource consumers
- Resource Usage
  - Top 10 CPU consumers
  - Top 10 Memory consumers
  - Processes using >80% CPU
  - Processes using >1GB RAM
- Problem Processes
  - Zombie processes (count + parents)
  - Defunct processes
  - Stuck in D state
  - Long-running processes (>7 days)
- Service Status
  - Failed services
  - Running critical services
  - Services with high resource usage
- Process Limits
  - Current vs max processes
  - File descriptor usage
  - Thread count
- Issues Found
  - Zombies to clean up
  - Services to restart
  - Resource hogs to investigate
  - Stuck processes
- Recommendations
  - Processes to kill/restart
  - Services to check
  - Limit adjustments needed
  - Monitoring suggestions

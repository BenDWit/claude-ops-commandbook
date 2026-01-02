# /linux-diagnostics-boot-system

Diagnose boot issues, systemd problems, and system startup failures.

## Usage
`/linux-diagnostics-boot-system`
`/linux-diagnostics-boot-system --failed-only`
`/linux-diagnostics-boot-system --last-boot`

## Tasks
1. Check systemd boot status and failed units
2. Analyze systemd journal for boot errors
3. Check dbus service status
4. Review boot time and slow services
5. Check for filesystem mount issues
6. Verify kernel boot parameters
7. Generate diagnostic report

## Systemd Status
```bash
# Overall system status
systemctl status

# List failed units
systemctl --failed

# Show all units and their status
systemctl list-units --all --state=failed

# Check specific critical services
systemctl status systemd-journald
systemctl status dbus
systemctl status NetworkManager
systemctl status sshd

# Boot time analysis
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain

# Time to userspace
systemd-analyze time
```

## Journal Analysis
```bash
# Boot messages from current boot
journalctl -b

# Boot messages from previous boot
journalctl -b -1

# Last 100 lines from current boot
journalctl -b -n 100

# Errors and critical from current boot
journalctl -b -p err..emerg

# Show only kernel messages
journalctl -k -b

# Follow boot log in real-time
journalctl -b -f

# Check for specific service failures
journalctl -u <service-name> -b

# All boots list
journalctl --list-boots
```

## DBus Diagnostics
```bash
# Check dbus service
systemctl status dbus

# Test dbus connectivity
dbus-send --system --print-reply --dest=org.freedesktop.DBus / org.freedesktop.DBus.Peer.Ping

# Check dbus socket
ls -l /var/run/dbus/system_bus_socket

# Monitor dbus activity
dbus-monitor --system

# Check for dbus errors in journal
journalctl -u dbus -b
```

## Filesystem Mount Issues
```bash
# Check all mounts
mount | column -t
df -h

# Verify /etc/fstab
cat /etc/fstab
findmnt --verify

# Check for mount failures
systemctl list-units --type=mount --all
journalctl -t mount -b

# Show failed mounts
systemctl --failed --type=mount
```

## Kernel and Boot Parameters
```bash
# Current kernel command line
cat /proc/cmdline

# Kernel ring buffer (boot messages)
dmesg | head -100
dmesg | grep -i error
dmesg | grep -i fail

# Check loaded kernel modules
lsmod | head -20

# Kernel version
uname -a
```

## GRUB and Bootloader
```bash
# Check GRUB configuration
cat /etc/default/grub

# List installed kernels
dpkg -l | grep linux-image  # Debian/Ubuntu
rpm -qa | grep kernel        # RHEL/CentOS

# Check GRUB menu entries
grep menuentry /boot/grub/grub.cfg | head -10

# Last boot log (if exists)
cat /var/log/boot.log 2>/dev/null || echo "Boot log not available"
```

## Hardware Initialization
```bash
# Check hardware detection
lspci
lsusb

# Check for firmware errors
journalctl -b | grep -i firmware

# ACPI errors
journalctl -b | grep -i acpi

# Check for disk errors during boot
journalctl -b | grep -i "i/o error"
```

## Service Dependencies
```bash
# Show dependencies for a service
systemctl list-dependencies <service>

# Show reverse dependencies (what depends on this)
systemctl list-dependencies --reverse <service>

# Check service startup order
systemctl list-jobs
```

## Emergency Recovery Info
```bash
# Boot into rescue mode (for reference)
# Add 'systemd.unit=rescue.target' to kernel parameters

# Boot into emergency mode
# Add 'systemd.unit=emergency.target' to kernel parameters

# Check if system is in degraded state
systemctl is-system-running

# Force service restart
systemctl restart <service>

# Reload systemd daemon
systemctl daemon-reload
```

## Output Format
Provide a diagnostic report with:
- Boot status (OK/Degraded/Failed)
- Failed units with error messages
- Boot time metrics
- Critical errors from journal
- DBus status
- Recommendations for fixing issues

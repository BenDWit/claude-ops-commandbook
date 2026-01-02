# /linux-diagnostics-disk-filesystem

Diagnose disk space, filesystem health, mount issues, and storage performance problems.

## Usage
`/linux-diagnostics-disk-filesystem`
`/linux-diagnostics-disk-filesystem --space`
`/linux-diagnostics-disk-filesystem --health`
`/linux-diagnostics-disk-filesystem --performance`

## Tasks
1. Check disk space and inode usage
2. Identify large files and directories
3. Check filesystem health and errors
4. Verify mount points and /etc/fstab
5. Analyze disk I/O performance
6. Check SMART status for drive health
7. Look for filesystem errors in logs
8. Generate storage diagnostic report

## Disk Space Analysis
```bash
# Disk usage overview
df -h
df -T  # Show filesystem types
df -i  # Inode usage

# Human-readable with filesystem type
df -hT

# Sort by usage
df -h | sort -k5 -rn

# Disk usage by directory (top level)
du -sh /* 2>/dev/null | sort -rh

# Find largest directories
du -h / 2>/dev/null | sort -rh | head -30
du -hx --max-depth=3 / 2>/dev/null | sort -rh | head -50

# Interactive disk usage browser
ncdu /  # if installed
```

## Large Files
```bash
# Find largest files (>100MB)
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -20

# Find largest files (>1GB)
find / -type f -size +1G -exec ls -lh {} \; 2>/dev/null

# Top 50 largest files
find / -type f -exec du -h {} \; 2>/dev/null | sort -rh | head -50

# Large files in specific directory
find /var -type f -size +50M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh

# Files larger than X in last 7 days
find / -type f -size +100M -mtime -7 -exec ls -lh {} \; 2>/dev/null

# Old large files (not accessed in 90 days)
find / -type f -size +100M -atime +90 -exec ls -lh {} \; 2>/dev/null
```

## Directory Usage
```bash
# Top directories by size
du -h / --max-depth=1 2>/dev/null | sort -rh | head -20

# Directories using most space in /var
du -sh /var/* 2>/dev/null | sort -rh

# Log directory sizes
du -sh /var/log/* | sort -rh

# Home directory usage
du -sh /home/* 2>/dev/null | sort -rh

# Count files per directory
for dir in /*; do
  echo -n "$dir: "
  find "$dir" -type f 2>/dev/null | wc -l
done | sort -t: -k2 -rn
```

## Inode Usage
```bash
# Inode usage per filesystem
df -i

# Find directories with many files
for i in /*; do
  echo "$i: $(find $i -type f 2>/dev/null | wc -l) files"
done | sort -t: -k2 -rn

# Find directory with most inodes used
find / -xdev -type d -exec bash -c 'echo "$(find "{}" -maxdepth 1 2>/dev/null | wc -l) {}"' \; 2>/dev/null | sort -rn | head -20
```

## Filesystem Health
```bash
# Check filesystem errors in dmesg
dmesg | grep -i "filesystem\|ext4\|xfs\|btrfs"
dmesg | grep -i "i/o error"

# Filesystem errors in journal
journalctl -k | grep -i "filesystem error"
journalctl -k | grep -i "ext4.*error"

# Check for read-only mounts
mount | grep "ro,"
findmnt -l | grep "ro,"

# Verify all mounts
mount | column -t
findmnt --verify

# Check /etc/fstab syntax
cat /etc/fstab
findmnt --verify --tab-file /etc/fstab

# Test mounting all fstab entries
mount -a --fake
```

## Disk Information
```bash
# List all block devices
lsblk
lsblk -f  # With filesystem info

# Detailed block device info
blkid
cat /proc/partitions

# Disk details
fdisk -l
parted -l

# Specific disk info
hdparm -I /dev/sda
hdparm -i /dev/sda

# UUID and filesystem
blkid | grep /dev/sd
```

## SMART Status (Drive Health)
```bash
# Check SMART capability
smartctl -i /dev/sda

# Overall health
smartctl -H /dev/sda

# Detailed SMART attributes
smartctl -a /dev/sda

# Run short self-test
smartctl -t short /dev/sda

# Check test results
smartctl -l selftest /dev/sda

# Error log
smartctl -l error /dev/sda

# For all drives
for disk in /dev/sd?; do
  echo "=== $disk ==="
  smartctl -H $disk 2>/dev/null
done
```

## Disk I/O Performance
```bash
# I/O statistics
iostat -x 1 5
iostat -d -x 1 5

# Per-device I/O
sar -d 1 5  # if sysstat installed

# I/O wait time
vmstat 1 5
# High 'wa' column indicates I/O bottleneck

# Process I/O
iotop -b -n 3  # if installed
pidstat -d 1 5

# Disk read/write test (careful!)
# Write test (creates 1GB file)
dd if=/dev/zero of=/tmp/testfile bs=1M count=1024 conv=fdatasync

# Read test
dd if=/tmp/testfile of=/dev/null bs=1M

# hdparm read test
hdparm -t /dev/sda
hdparm -T /dev/sda

# fio benchmark (if installed)
fio --name=random-write --ioengine=libaio --iodepth=16 --rw=randwrite --bs=4k --size=1G --numjobs=1 --runtime=60 --time_based --end_fsync=1
```

## Mount Issues
```bash
# Currently mounted filesystems
mount | column -t
findmnt

# Failed mounts
systemctl --failed --type=mount
journalctl -t mount | tail -50

# Mount options
mount | grep -E "rw,|ro,"

# Check if filesystem is mounted
mountpoint /mnt/data

# List NFS mounts
mount | grep nfs
showmount -e localhost

# Check for stale NFS mounts
df -h  # Will hang on stale mounts
timeout 5 df -h
```

## Filesystem Specific Commands

### ext4
```bash
# Check ext4 filesystem
e2fsck -n /dev/sda1  # Dry run (no changes)
# NEVER run e2fsck on mounted filesystem!

# Filesystem info
tune2fs -l /dev/sda1

# Reserved blocks
tune2fs -l /dev/sda1 | grep "Reserved block count"

# Last check time
tune2fs -l /dev/sda1 | grep "Last checked"

# Defrag check (fragmentation %)
e4defrag -c /
```

### XFS
```bash
# XFS filesystem info
xfs_info /dev/sda1

# Check filesystem (unmounted only!)
xfs_repair -n /dev/sda1  # Dry run

# XFS fragmentation
xfs_db -c frag -r /dev/sda1
```

### btrfs
```bash
# Btrfs filesystem usage
btrfs filesystem usage /
btrfs filesystem df /

# Btrfs device stats
btrfs device stats /

# Scrub status
btrfs scrub status /
```

## Deleted But Open Files
```bash
# Files deleted but still held open (using disk space)
lsof +L1

# Find processes holding deleted files
lsof | grep deleted

# Total space held by deleted files
lsof +L1 | awk '{sum+=$7} END {print sum/1024/1024 " MB"}'
```

## Log File Management
```bash
# Log directory sizes
du -sh /var/log/* | sort -rh

# Large log files
find /var/log -type f -size +100M -exec ls -lh {} \;

# Compress old logs
find /var/log -name "*.log" -type f -mtime +7 -exec gzip {} \;

# Clean systemd journal
journalctl --vacuum-time=7d
journalctl --vacuum-size=500M

# Journal disk usage
journalctl --disk-usage

# Truncate large log (carefully!)
truncate -s 0 /var/log/large.log
# Or: > /var/log/large.log
```

## Cache and Temporary Files
```bash
# Check cache usage
du -sh /var/cache/* | sort -rh

# Temporary file usage
du -sh /tmp /var/tmp

# Clear package cache
# Debian/Ubuntu:
apt-get clean
du -sh /var/cache/apt

# RHEL/CentOS:
yum clean all
dnf clean all

# Old temp files
find /tmp -type f -atime +7 -exec ls -lh {} \;
```

## Quota Management
```bash
# Check if quotas are enabled
mount | grep quota
cat /etc/fstab | grep quota

# User quotas
quota -u <username>
repquota -a

# Group quotas
quota -g <groupname>
repquota -ag

# Quota report
repquota -s /home
```

## LVM Diagnostics
```bash
# Physical volumes
pvdisplay
pvs

# Volume groups
vgdisplay
vgs

# Logical volumes
lvdisplay
lvs

# LVM space usage
vgs -o +vg_free
lvs -o +lv_size,lv_free

# Extend filesystem (example)
# lvextend -L +10G /dev/vg/lv
# resize2fs /dev/vg/lv  # for ext4
# xfs_growfs /mount/point  # for XFS
```

## RAID Status
```bash
# Software RAID status
cat /proc/mdstat

# Detailed RAID info
mdadm --detail /dev/md0

# Check for failed drives
mdadm --detail /dev/md0 | grep -E "State|Failed"

# RAID events
dmesg | grep -i raid
journalctl | grep -i raid
```

## Filesystem Errors in Logs
```bash
# Check kernel logs for disk errors
dmesg | grep -i "error\|fail" | grep -i "sd\|hd"

# Journal filesystem errors
journalctl -k | grep -i "filesystem\|ext4\|xfs"

# I/O errors
journalctl -k | grep -i "i/o error"

# Disk errors
grep -i "error" /var/log/syslog | grep -i "disk\|sd"
```

## Recovery Commands
```bash
# Force filesystem check on next boot (ext4)
touch /forcefsck

# Schedule fsck (ext4)
tune2fs -C 1 /dev/sda1  # Will run on next boot

# Emergency read-only remount
mount -o remount,ro /

# Emergency read-write remount
mount -o remount,rw /

# Clear deleted space (zero out deleted files)
sfill -f /mount/point  # if secure-delete installed
```

## Monitoring Setup
```bash
# Set up disk monitoring alerts
# Configure smartd
systemctl enable smartd
systemctl start smartd
cat /etc/smartd.conf

# Set up disk space alerts (example cron job)
# Add to crontab:
# 0 * * * * df -h | grep -E "9[0-9]%|100%" && echo "Disk space critical" | mail -s "Disk Alert" admin@example.com
```

## Output Format
Provide a disk/filesystem diagnostic report with:
- Disk Space Summary
  - Per-filesystem usage %
  - Filesystems >80% full (warning)
  - Filesystems >90% full (critical)
  - Inode usage
- Large Files/Directories
  - Top 10 largest files
  - Top 10 directories by size
  - Deleted but open files
- Filesystem Health
  - Mount status (all/failed)
  - Filesystem errors found
  - SMART status per disk
  - Read-only mounts
- Performance
  - I/O wait time
  - Slowest disks
  - RAID status (if applicable)
- Issues Found
  - Space exhaustion
  - Inode exhaustion
  - Filesystem errors
  - Failed mounts
  - Disk hardware issues
- Recommendations
  - Files/logs to clean up
  - Filesystems to expand
  - Health checks needed
  - Monitoring suggestions

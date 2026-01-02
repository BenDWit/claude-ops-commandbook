# /plesk-backup

Create and manage Plesk backups for domains or full server.

## Usage
`/plesk-backup example.com`
`/plesk-backup example.com --full`
`/plesk-backup --server`
`/plesk-backup --list example.com`

## Tasks
1. Determine backup scope (domain/subscription/server)
2. Check available disk space
3. Create backup with appropriate settings
4. Verify backup integrity
5. Report backup location and size

## Commands
```bash
# Backup single domain (files + databases + mail)
plesk bin pleskbackup domains-name example.com -v --output-file=/var/lib/psa/dumps/example.com_$(date +%Y%m%d).tar

# Backup subscription (includes all domains under customer)
plesk bin pleskbackup subscription example.com -v

# Full server backup
plesk bin pleskbackup server -v

# Incremental backup
plesk bin pleskbackup domains-name example.com --incremental

# Backup to FTP
plesk bin pleskbackup domains-name example.com --ftp-login=user --ftp-password=pass --ftp-address=ftp.backup.com

# List existing backups
plesk bin pleskrestore --list

# List backups for specific domain
plesk bin pleskrestore --list -name example.com
```

## Pre-backup Checks
```bash
# Check disk space
df -h /var/lib/psa/dumps

# Estimate domain size
du -sh /var/www/vhosts/example.com/

# Check database sizes
plesk db "SELECT table_schema, ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)' FROM information_schema.tables GROUP BY table_schema;"
```

## Restore Commands
```bash
# List contents of backup
plesk bin pleskrestore --info -level 3 backup_file.tar

# Restore domain
plesk bin pleskrestore --restore backup_file.tar -level domains -filter example.com

# Restore specific components
plesk bin pleskrestore --restore backup_file.tar -level domains -filter example.com -content files
plesk bin pleskrestore --restore backup_file.tar -level domains -filter example.com -content db
```

## Backup Locations
- Default: `/var/lib/psa/dumps/`
- Custom: Specified with `--output-file`
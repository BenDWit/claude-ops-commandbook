# /plesk-db

Manage databases and database users in Plesk.

## Usage
`/plesk-db example.com --list`
`/plesk-db example.com --create mydb`
`/plesk-db example.com --size`
`/plesk-db example.com --backup mydb`
`/plesk-db example.com --repair mydb`

## Tasks
1. List databases for domain
2. Create/delete databases and users
3. Check database sizes
4. Backup/restore databases
5. Repair corrupted tables
6. Optimize databases

## List and Info
```bash
# List databases for domain
plesk bin database --list example.com

# Database details
plesk bin database --info dbname

# List all databases on server
plesk db "SHOW DATABASES;"

# List database users
plesk bin database --list-users example.com
```

## Database Sizes
```bash
# All database sizes
plesk db "SELECT table_schema AS 'Database', 
  ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)' 
  FROM information_schema.tables 
  GROUP BY table_schema 
  ORDER BY SUM(data_length + index_length) DESC;"

# Specific database size
plesk db "SELECT table_name, 
  ROUND(data_length/1024/1024, 2) AS 'Data (MB)',
  ROUND(index_length/1024/1024, 2) AS 'Index (MB)'
  FROM information_schema.tables 
  WHERE table_schema='dbname'
  ORDER BY data_length DESC;"

# Find large tables
plesk db "SELECT table_schema, table_name, 
  ROUND((data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
  FROM information_schema.tables 
  ORDER BY (data_length + index_length) DESC 
  LIMIT 20;"
```

## Create Database
```bash
# Create database
plesk bin database --create mydb -domain example.com -type mysql

# Create database user
plesk bin database --create-dbuser myuser -passwd 'StrongPass123!' -database mydb -domain example.com

# Create database with user in one go
plesk bin database --create mydb -domain example.com -type mysql \
  -add_user myuser -passwd 'StrongPass123!'
```

## Backup and Restore
```bash
# Backup single database
mysqldump -u admin -p$(cat /etc/psa/.psa.shadow) dbname > /tmp/dbname_backup.sql

# Backup with compression
mysqldump -u admin -p$(cat /etc/psa/.psa.shadow) dbname | gzip > /tmp/dbname_$(date +%Y%m%d).sql.gz

# Backup all databases
mysqldump -u admin -p$(cat /etc/psa/.psa.shadow) --all-databases > /tmp/all_databases.sql

# Restore database
mysql -u admin -p$(cat /etc/psa/.psa.shadow) dbname < /tmp/dbname_backup.sql

# Restore from gzip
gunzip < /tmp/dbname_backup.sql.gz | mysql -u admin -p$(cat /etc/psa/.psa.shadow) dbname
```

## Repair and Optimize
```bash
# Check all tables in database
mysqlcheck -u admin -p$(cat /etc/psa/.psa.shadow) --check dbname

# Repair all tables
mysqlcheck -u admin -p$(cat /etc/psa/.psa.shadow) --repair dbname

# Optimize all tables (reclaim space)
mysqlcheck -u admin -p$(cat /etc/psa/.psa.shadow) --optimize dbname

# Check all databases
mysqlcheck -u admin -p$(cat /etc/psa/.psa.shadow) --all-databases --check

# Repair specific table
plesk db "REPAIR TABLE dbname.tablename;"
```

## User Management
```bash
# Reset database user password
plesk bin database --update-dbuser myuser -passwd 'NewStrongPass123!'

# Grant remote access to user
plesk db "GRANT ALL PRIVILEGES ON dbname.* TO 'myuser'@'%' IDENTIFIED BY 'password';"
plesk db "FLUSH PRIVILEGES;"

# Show user privileges
plesk db "SHOW GRANTS FOR 'myuser'@'localhost';"

# Delete database user
plesk bin database --remove-dbuser myuser
```

## Quick Queries
```bash
# Connect to MySQL CLI
plesk db

# Run query directly
plesk db "SELECT * FROM dbname.wp_options LIMIT 5;"

# Count rows in table
plesk db "SELECT COUNT(*) FROM dbname.tablename;"

# Show running queries
plesk db "SHOW PROCESSLIST;"

# Kill long-running query
plesk db "KILL <process_id>;"
```

## WordPress-Specific
```bash
# Find WordPress database
grep DB_NAME /var/www/vhosts/example.com/httpdocs/wp-config.php

# Change WordPress URL
plesk db "UPDATE dbname.wp_options SET option_value='https://newdomain.com' WHERE option_name='siteurl';"
plesk db "UPDATE dbname.wp_options SET option_value='https://newdomain.com' WHERE option_name='home';"

# Reset WordPress admin password
plesk db "UPDATE dbname.wp_users SET user_pass=MD5('newpassword') WHERE user_login='admin';"
```

## Troubleshooting
```bash
# Check MySQL status
systemctl status mariadb

# Check MySQL error log
tail -50 /var/log/mysql/error.log

# Check connections
plesk db "SHOW STATUS LIKE 'Threads_connected';"

# Check slow queries
tail -50 /var/log/mysql/slow.log
```
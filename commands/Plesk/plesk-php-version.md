# /plesk-php

Manage PHP versions and settings for domains.

## Usage
`/plesk-php example.com --version 8.2`
`/plesk-php example.com --info`
`/plesk-php --list-versions`
`/plesk-php example.com --setting memory_limit=512M`

## Tasks
1. Check current PHP version and handler
2. List available PHP versions
3. Change PHP version if requested
4. Modify PHP settings as needed
5. Verify changes and test

## Commands
```bash
# List available PHP versions
plesk bin php_handler --list

# Get current PHP settings for domain
plesk bin domain --info example.com | grep -i php

# Get detailed PHP settings
plesk bin site --info example.com -php

# Change PHP version
plesk bin site --update example.com -php_handler_id plesk-php82-fpm

# Change PHP handler type (FPM vs CGI)
plesk bin site --update example.com -php_handler_type fpm

# Update PHP settings
plesk bin site --update-php-settings example.com -settings "memory_limit=512M;max_execution_time=300;post_max_size=128M;upload_max_filesize=128M"
```

## Common PHP Settings
```bash
# Increase memory limit
plesk bin site --update-php-settings example.com -settings "memory_limit=512M"

# Increase upload limits (for WordPress etc)
plesk bin site --update-php-settings example.com -settings "upload_max_filesize=256M;post_max_size=256M"

# Increase execution time
plesk bin site --update-php-settings example.com -settings "max_execution_time=600"

# Enable/disable functions
plesk bin site --update-php-settings example.com -settings "disable_functions="
```

## PHP Handler Types
- **FPM** (recommended): Best performance, process-based
- **CGI**: Legacy, slower but isolated
- **FastCGI**: Balance of performance and isolation

## Install Additional PHP Version
```bash
# List available for install (Debian/Ubuntu)
plesk installer --select-release-current --show-components | grep php

# Install PHP 8.3
plesk installer --select-release-current --install-component php8.3

# Enable PHP-FPM for new version
plesk bin php_handler --reread
```

## Troubleshooting
```bash
# Check PHP-FPM status
systemctl status plesk-php82-fpm

# Check PHP error log
tail -50 /var/www/vhosts/example.com/logs/error_log

# Test PHP configuration
php -v
php -i | grep memory_limit

# Restart PHP-FPM
systemctl restart plesk-php82-fpm
```
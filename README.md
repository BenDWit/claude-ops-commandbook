# Claude Ops Commandbook

A collection of custom slash commands for Claude Code, providing structured workflows for server administration and filesystem management tasks.

## Table of Contents

- [Claude Ops Commandbook](#claude-ops-commandbook)
  - [Table of Contents](#table-of-contents)
  - [Installation](#installation)
  - [Available Commands](#available-commands)
    - [Filesystem](#filesystem)
    - [Plesk](#plesk)
  - [Usage Examples](#usage-examples)
    - [Filesystem Commands](#filesystem-commands)
    - [Plesk Commands](#plesk-commands)
  - [Requirements](#requirements)
  - [Notes](#notes)
  - [Customization](#customization)

## Installation

Copy the `commands/` directory to your Claude Code commands folder:

```bash
# Default location
cp -r commands/ ~/.claude/commands/

# Or project-specific
cp -r commands/ .claude/commands/
```

## Available Commands

### Filesystem

Commands for filesystem-related tasks, such as cleanup and organization.

| Command | Description |
|---------|-------------|
| `/filesystem-cleanup-desktop-tidy` | Organize and clean up files on the desktop |
| `/filesystem-cleanup-download-tidy` | Organize and clean up files in the downloads folder |

### Plesk

Commands for Plesk server administration.

| Command | Description |
|---------|-------------|
| `/plesk-backup` | Create and manage backups |
| `/plesk-databases` | Database management and troubleshooting |
| `/plesk-domain-status` | Check domain configuration (hosting, SSL, DNS, mail) |
| `/plesk-logs` | View and analyze logs |
| `/plesk-mail-debug` | Diagnose mail delivery issues |
| `/plesk-performance` | Performance analysis and optimization |
| `/plesk-php-version` | Manage PHP versions and settings |
| `/plesk-security-audit` | Perform security audits |
| `/plesk-ssl-renew` | Renew/install Let's Encrypt SSL certificates |

## Usage Examples

### Filesystem Commands

```bash
# Organize desktop files
/filesystem-cleanup-desktop-tidy

# Clean up downloads folder
/filesystem-cleanup-download-tidy
```

### Plesk Commands

```bash
# Check domain status
/plesk-domain-status example.com

# Renew SSL with www subdomain
/plesk-ssl-renew example.com --include-www

# Create backup
/plesk-backup example.com

# Change PHP version
/plesk-php-version example.com --version 8.2

# Security audit
/plesk-security-audit

# Debug mail issues
/plesk-mail-debug user@example.com

# Database operations
/plesk-databases example.com --list
/plesk-databases example.com --backup mydb

# View logs
/plesk-logs example.com --error

# Performance check
/plesk-performance example.com
```

## Requirements

- **Filesystem Commands**: Basic shell access and file permissions
- **Plesk Commands**: Root/sudo access to Plesk server, SSH connection, Plesk CLI tools (`plesk bin`)

## Notes

- Commands assume a Linux environment (Debian/Ubuntu for Plesk-specific commands)
- Some Plesk commands may need adjustment for CentOS/RHEL installations
- Always test commands in a non-production environment first
- Backup important data before making changes
- These commands are designed to work with Claude Code's slash command system

## Customization

Feel free to modify these commands for your specific environment:

- Add company-specific paths or configurations
- Include custom monitoring endpoints
- Add integration with your ticketing or notification systems
- Extend with additional Plesk extensions or filesystem utilities
- Adapt for different operating systems or server setups
# Claude Ops Commandbook

A collection of custom slash commands for Claude Code, providing structured workflows for server administration and filesystem management tasks.

## Table of Contents

- [Claude Ops Commandbook](#claude-ops-commandbook)
  - [Table of Contents](#table-of-contents)
  - [Installation](#installation)
  - [Available Commands](#available-commands)
    - [Filesystem](#filesystem)
    - [Linux Diagnostics](#linux-diagnostics)
    - [Monitoring Diagnostics](#monitoring-diagnostics)
    - [Plesk](#plesk)
  - [Usage Examples](#usage-examples)
    - [Filesystem Commands](#filesystem-commands)
    - [Linux Diagnostics Commands](#linux-diagnostics-commands)
    - [Monitoring Diagnostics Commands](#monitoring-diagnostics-commands)
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

### Linux Diagnostics

Comprehensive system diagnostic commands for Linux servers.

| Command | Description |
|---------|-------------|
| `/linux-diagnostics-boot-system` | Diagnose boot issues, systemd, and dbus problems |
| `/linux-diagnostics-disk-filesystem` | Check disk space, filesystem health, and storage issues |
| `/linux-diagnostics-network` | Diagnose network connectivity, DNS, routing, and firewall |
| `/linux-diagnostics-performance` | Analyze CPU, memory, disk I/O, and resource bottlenecks |
| `/linux-diagnostics-process` | Troubleshoot processes, zombies, and resource hogs |
| `/linux-diagnostics-security-audit` | Audit sudo, users, SSH, and authentication security |
| `/linux-diagnostics-webserver` | Diagnose Apache/Nginx webserver issues and logs |

### Monitoring Diagnostics

Diagnostic commands for monitoring systems like Grafana and Wazuh.

| Command | Description |
|---------|-------------|
| `/monitoring-diagnostics-grafana` | Diagnose Grafana service health, datasource connectivity, dashboards, and alerts |
| `/monitoring-diagnostics-wazuh` | Diagnose Wazuh manager/agent status, alerts, rules, and log processing |

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

### Linux Diagnostics Commands

```bash
# Check boot and systemd issues
/linux-diagnostics-boot-system

# Analyze system performance
/linux-diagnostics-performance

# Network diagnostics
/linux-diagnostics-network

# Security audit
/linux-diagnostics-security-audit

# Disk and filesystem check
/linux-diagnostics-disk-filesystem

# Process diagnostics
/linux-diagnostics-process --high-cpu

# Webserver diagnostics
/linux-diagnostics-webserver --apache
```

### Monitoring Diagnostics Commands

```bash
# Grafana full diagnostics
/monitoring-diagnostics-grafana

# Check datasources only
/monitoring-diagnostics-grafana --datasources

# Check dashboard errors
/monitoring-diagnostics-grafana --dashboards

# Wazuh manager and agents
/monitoring-diagnostics-wazuh

# Cluster health check
/monitoring-diagnostics-wazuh --cluster

# Agent status only
/monitoring-diagnostics-wazuh --agents
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
- **Linux Diagnostics Commands**: Root/sudo access to Linux server, SSH connection, standard diagnostic tools (may require installing sysstat, smartmontools, etc.)
- **Monitoring Diagnostics Commands**:
  - Grafana: Access to Grafana server (SSH), API token for full diagnostics, port 3000 access
  - Wazuh: Root/sudo access to Wazuh manager, access to `/var/ossec/` directory, API authentication
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
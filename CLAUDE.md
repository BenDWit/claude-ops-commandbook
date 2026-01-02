# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of custom slash commands for Claude Code, providing structured workflows for server administration (Plesk) and filesystem management tasks. These are markdown-based command definitions that guide Claude Code through complex operational procedures.

## Repository Structure

```
commands/
├── Plesk/              # Plesk server administration commands
│   ├── plesk-backup.md
│   ├── plesk-databases.md
│   ├── PLesk-domain-status.md
│   ├── plesk-logs.md
│   ├── plesk-mail-debug.md
│   ├── Plesk-performance.md
│   ├── plesk-php-version.md (command: /plesk-php)
│   ├── plesk-security-audit.md
│   └── Plesk-ssl-renew.md
└── filesystem/
    └── cleanup/        # Filesystem organization commands
        ├── desktop-tidy.md
        └── download-tidy.md
```

## Command Structure Pattern

All command files follow a consistent structure:

1. **Header**: Command name (e.g., `# /plesk-backup`)
2. **Description**: Brief summary of what the command does
3. **Usage**: Example invocations with parameters
4. **Tasks**: Numbered step-by-step workflow Claude should follow
5. **Commands**: Bash command snippets organized by purpose
6. **Additional Sections**: Domain-specific information (e.g., "Troubleshooting", "Common Issues", "Output Format")

### Command Types

**Plesk Commands**: Structured, technical commands with:
- Specific `plesk bin` CLI commands
- Pre-check procedures (disk space, existing configurations)
- Troubleshooting sections with diagnostic commands
- Common issues and solutions

**Filesystem Commands**: Conversational, user-interactive commands that:
- Work with user directories (`~/Desktop`, `~/Downloads`)
- Organize files by type into folders
- Prompt user before deleting potentially unused files
- Are less structured and more interactive

## Installation

Commands are meant to be copied to Claude Code's commands directory:
- Default: `~/.claude/commands/`
- Project-specific: `.claude/commands/`

## Key Design Principles

1. **Commands are Templates**: Each `.md` file is a guide for Claude to execute, not executable scripts
2. **Consistent Naming**: Command names use the pattern `/{category}-{action}` (e.g., `/plesk-backup`, `/filesystem-cleanup-desktop-tidy`)
3. **Note Filename Inconsistencies**: Some files have inconsistent capitalization (e.g., `PLesk-domain-status.md` vs `plesk-backup.md`)
4. **Environment Assumptions**:
   - Linux environment (Debian/Ubuntu for Plesk commands)
   - Plesk commands require root/sudo access and SSH
   - Plesk CLI tools (`plesk bin`) must be available

## Working with This Repository

### Adding New Commands

1. Choose appropriate category directory (`Plesk/` or `filesystem/cleanup/`)
2. Create `.md` file with kebab-case naming
3. Follow the standard structure (Header → Usage → Tasks → Commands)
4. Include troubleshooting section for complex commands
5. Update README.md table with new command

### Modifying Commands

- Preserve the Tasks section structure (numbered workflow steps)
- Keep command examples up-to-date with Plesk CLI versions
- Maintain separation between "happy path" commands and troubleshooting

### Testing Considerations

- Plesk commands should be tested on non-production servers first
- Commands assume specific directory structures (`/var/www/vhosts/`, `/var/lib/psa/dumps/`)
- Some commands may need adjustment for CentOS/RHEL (currently Debian/Ubuntu focused)

## Common Plesk CLI Patterns

```bash
# Domain info: plesk bin domain --info <domain>
# SSL management: plesk bin certificate --list/--info -domain <domain>
# Site config: plesk bin site --update <domain> [options]
# Server preferences: plesk bin server_pref --show/--update
# Extensions: plesk bin extension --exec <extension> [args]
```

## Important Notes

- **Case Sensitivity**: Some command filenames have inconsistent capitalization (this should be normalized)
- **No Build/Test Commands**: This is a documentation repository with no compilation or testing required
- **Git Workflow**: Standard git operations (commit messages should reference which command was added/modified)

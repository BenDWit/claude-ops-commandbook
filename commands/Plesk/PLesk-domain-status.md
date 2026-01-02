# /plesk-domain-status

Check the status of a domain in Plesk including hosting, DNS, SSL, and mail configuration.

## Usage
`/plesk-domain-status example.com`

## Tasks
1. Verify domain exists in Plesk: `plesk bin domain --info <domain>`
2. Check hosting status and document root
3. Check SSL certificate status: `plesk bin certificate --info -domain <domain>`
4. Verify DNS zone: `plesk bin dns --info <domain>`
5. Check mail configuration: `plesk bin mail --info <domain>`
6. Report any issues found

## Example Commands
```bash
# Full domain info
plesk bin domain --info example.com

# SSL status
plesk bin certificate --list -domain example.com

# DNS records
plesk bin dns --info example.com

# Mail status
plesk bin mail --info example.com

# Webspace owner info
plesk bin subscription --info example.com
```

## Output Format
Provide a summary table with:
- Domain name
- Hosting status (active/suspended)
- SSL status (valid/expired/missing)
- SSL expiry date
- DNS status
- Mail enabled (yes/no)
- PHP version
- Document root
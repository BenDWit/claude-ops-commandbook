# /plesk-ssl-renew

Renew or install Let's Encrypt SSL certificate for a domain.

## Usage
`/plesk-ssl-renew example.com`
`/plesk-ssl-renew example.com --include-www`
`/plesk-ssl-renew example.com --wildcard`

## Tasks
1. Check current certificate status
2. Verify domain resolves correctly (DNS propagation)
3. Issue/renew Let's Encrypt certificate
4. Verify installation
5. Check for common issues (rate limits, DNS problems)

## Commands
```bash
# Check current cert
plesk bin certificate --list -domain example.com

# Issue Let's Encrypt (standard)
plesk bin extension --exec letsencrypt cli.php -d example.com -m admin@example.com

# Issue with www subdomain included
plesk bin extension --exec letsencrypt cli.php -d example.com -d www.example.com -m admin@example.com

# Wildcard certificate (requires DNS validation)
plesk bin extension --exec letsencrypt cli.php -d example.com -d "*.example.com" -m admin@example.com --dns

# Force renewal (if not expiring soon)
plesk bin extension --exec letsencrypt cli.php -d example.com --force

# Check rate limits
curl -s https://crt.sh/?q=example.com | grep -c "example.com"
```

## Troubleshooting
```bash
# Check Let's Encrypt logs
tail -100 /var/log/plesk/ext-letsencrypt.log

# Verify DNS resolution
dig +short example.com
dig +short www.example.com

# Test HTTP validation path accessibility
curl -I http://example.com/.well-known/acme-challenge/test

# Check if behind proxy/CDN (may need DNS validation)
curl -sI example.com | grep -i server
```

## Common Issues
- **Rate limited**: Wait 1 hour or use staging environment
- **DNS not propagated**: Wait for TTL, verify with `dig`
- **Behind Cloudflare**: Use DNS validation or pause Cloudflare proxy
- **Firewall blocking**: Ensure port 80 is open for HTTP validation
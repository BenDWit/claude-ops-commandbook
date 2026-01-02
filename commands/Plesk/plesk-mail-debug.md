# /plesk-mail-debug

Diagnose and troubleshoot mail issues for a domain.

## Usage
`/plesk-mail-debug example.com`
`/plesk-mail-debug user@example.com`
`/plesk-mail-debug example.com --queue`
`/plesk-mail-debug example.com --auth-test`

## Tasks
1. Check mail service status
2. Verify DNS records (MX, SPF, DKIM, DMARC)
3. Check mailbox status and quota
4. Review mail logs for errors
5. Test authentication
6. Check mail queue

## Mail Service Status
```bash
# Check mail services
systemctl status postfix
systemctl status dovecot
systemctl status plesk-ext-mailman  # if using mailing lists

# Check mail configuration for domain
plesk bin mail --info example.com

# List mailboxes
plesk bin mail --list example.com

# Mailbox info
plesk bin mail --info user@example.com
```

## DNS Records Check
```bash
# MX records
dig +short MX example.com

# SPF record
dig +short TXT example.com | grep spf

# DKIM record
dig +short TXT default._domainkey.example.com

# DMARC record
dig +short TXT _dmarc.example.com

# Verify Plesk DKIM is enabled
plesk bin domain --info example.com | grep -i dkim
```

## Mail Queue
```bash
# Check mail queue
postqueue -p

# Queue count
postqueue -p | tail -1

# Flush queue (send pending)
postqueue -f

# Delete all queued mail (DANGEROUS)
# postsuper -d ALL

# Delete specific message
postsuper -d <message-id>

# View specific message
postcat -q <message-id>
```

## Log Analysis
```bash
# Recent mail activity
tail -100 /var/log/maillog

# Failed deliveries
grep -i "status=bounced\|status=deferred" /var/log/maillog | tail -20

# Authentication failures
grep -i "auth.*fail" /var/log/maillog | tail -20

# Specific domain/user
grep "example.com" /var/log/maillog | tail -50
grep "user@example.com" /var/log/maillog | tail -50

# Connection issues
grep -i "connect.*fail\|timeout" /var/log/maillog | tail -20
```

## Authentication Test
```bash
# Test SMTP auth (requires swaks)
swaks --to test@gmail.com --from user@example.com --server mail.example.com --auth LOGIN --auth-user user@example.com --auth-password "password" --tls

# Test IMAP connection
openssl s_client -connect mail.example.com:993

# Check SSL certificate for mail
openssl s_client -connect mail.example.com:465 -servername mail.example.com 2>/dev/null | openssl x509 -noout -dates
```

## Quota and Storage
```bash
# Check mailbox quota
plesk bin mail --info user@example.com | grep -i quota

# Actual mailbox size
du -sh /var/qmail/mailnames/example.com/user/

# Set quota
plesk bin mail --update user@example.com -mailbox true -mbox_quota 1024M
```

## Common Fixes
```bash
# Reset mailbox password
plesk bin mail --update user@example.com -passwd "newpassword"

# Enable/disable mailbox
plesk bin mail --update user@example.com -mailbox true
plesk bin mail --update user@example.com -mailbox false

# Enable DKIM
plesk bin domain --update example.com -dkim true

# Restart mail services
systemctl restart postfix dovecot

# Regenerate mail configuration
plesk repair mail -y
```

## Blacklist Check
```bash
# Check if IP is blacklisted (basic)
SERVER_IP=$(hostname -I | awk '{print $1}')
echo "Check these manually:"
echo "- https://mxtoolbox.com/blacklists.aspx?q=$SERVER_IP"
echo "- https://www.spamhaus.org/lookup/"

# Reverse DNS check
dig +short -x $SERVER_IP
```

## Output Format
Provide diagnosis with:
- Mail service status (running/stopped)
- DNS records status (correct/missing/incorrect)
- Recent errors (last 5)
- Queue status (count, oldest message age)
- Recommended actions
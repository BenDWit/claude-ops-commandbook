# /linux-diagnostics-network

Diagnose network connectivity, DNS, routing, firewall, and performance issues.

## Usage
`/linux-diagnostics-network`
`/linux-diagnostics-network --connectivity`
`/linux-diagnostics-network --dns`
`/linux-diagnostics-network --firewall`

## Tasks
1. Check network interface status
2. Test connectivity and routing
3. Verify DNS resolution
4. Check firewall rules
5. Analyze active connections
6. Test network performance
7. Check for security issues
8. Generate network diagnostic report

## Network Interface Status
```bash
# List all interfaces
ip link show
ip addr show
ifconfig -a

# Interface statistics
ip -s link
ifconfig -s

# Network interface details
nmcli device status
nmcli connection show

# Check interface up/down
ip link show | grep -E "^[0-9]|state"

# MTU settings
ip link | grep mtu

# Hardware info
ethtool eth0  # Replace with your interface
lshw -class network
```

## IP Configuration
```bash
# Current IP addresses
ip addr show
hostname -I
hostname -i

# Gateway/default route
ip route show
ip route get 8.8.8.8
route -n

# ARP table (MAC addresses)
ip neigh show
arp -n

# Check for IP conflicts
arping -D -I eth0 -c 2 <IP_ADDRESS>
```

## Connectivity Testing
```bash
# Test internet connectivity
ping -c 4 8.8.8.8
ping -c 4 google.com

# Test specific host
ping -c 4 <hostname>

# Traceroute to destination
traceroute google.com
traceroute -n 8.8.8.8  # Skip DNS resolution
mtr google.com  # if installed (better than traceroute)

# Test specific port connectivity
nc -zv <host> <port>
telnet <host> <port>
timeout 5 bash -c "</dev/tcp/<host>/<port>" && echo "Port open" || echo "Port closed"

# Check gateway reachability
ping -c 4 $(ip route | grep default | awk '{print $3}')

# Path MTU discovery
ping -M do -s 1472 -c 4 google.com
```

## DNS Diagnostics
```bash
# Check DNS resolution
nslookup google.com
dig google.com
host google.com

# Check specific DNS server
nslookup google.com 8.8.8.8
dig @8.8.8.8 google.com

# Reverse DNS lookup
dig -x <IP_ADDRESS>
nslookup <IP_ADDRESS>

# DNS configuration
cat /etc/resolv.conf

# systemd-resolved (Ubuntu/newer systems)
systemd-resolve --status
resolvectl status

# Test DNS performance
dig google.com | grep "Query time"

# Check all DNS record types
dig google.com ANY
dig google.com A
dig google.com MX
dig google.com NS

# Flush DNS cache
systemd-resolve --flush-caches  # systemd-resolved
resolvectl flush-caches         # newer systemd
systemctl restart nscd          # nscd
systemctl restart dnsmasq       # dnsmasq

# Check if DNS is working vs internet
ping -c 2 8.8.8.8 && echo "Internet works" || echo "No internet"
ping -c 2 google.com && echo "DNS works" || echo "DNS issue"
```

## Active Connections
```bash
# All TCP/UDP connections
ss -tuln
ss -tulpn  # With process names (needs root)
netstat -tuln
netstat -tulpn

# Established connections
ss -tn state established
netstat -tn | grep ESTABLISHED

# Listening ports
ss -tln
ss -uln
lsof -i -P -n | grep LISTEN

# Connections by state
ss -tan | awk '{print $1}' | sort | uniq -c
netstat -an | awk '{print $6}' | sort | uniq -c

# Connections to specific port
ss -tn sport = :80
ss -tn dport = :443

# Top connections by IP
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -20

# Count connections per IP
ss -tn | grep ESTAB | awk '{print $4}' | cut -d: -f1 | sort | uniq -c | sort -rn
```

## Port Scanning
```bash
# Scan localhost
nmap localhost
nmap -sT localhost

# Scan specific host
nmap <hostname>
nmap -p 1-1000 <hostname>

# Check if port is open
nmap -p 22,80,443 <hostname>

# Service detection
nmap -sV <hostname>

# Quick ping scan of network
nmap -sn 192.168.1.0/24
```

## Firewall Diagnostics
```bash
# UFW (Ubuntu)
ufw status verbose
ufw status numbered
ufw show added

# firewalld (RHEL/CentOS)
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --list-services
firewall-cmd --list-ports

# iptables
iptables -L -n -v
iptables -L INPUT -n -v
iptables -L OUTPUT -n -v
iptables -L FORWARD -n -v
iptables -t nat -L -n -v

# Count rules per chain
iptables -L -n | grep -E "^Chain" -A 100

# nftables (newer systems)
nft list ruleset

# Check if packet filtering is enabled
cat /proc/sys/net/ipv4/ip_forward
```

## Network Performance
```bash
# Bandwidth usage by interface
sar -n DEV 1 5  # if sysstat installed
ifstat -t 1 5   # if installed

# Real-time bandwidth monitor
iftop  # if installed
iftop -i eth0

# Bandwidth by process
nethogs  # if installed

# Network statistics
netstat -s
ss -s

# TCP window scaling
sysctl net.ipv4.tcp_window_scaling

# Network buffer sizes
sysctl net.core.rmem_max
sysctl net.core.wmem_max
sysctl net.ipv4.tcp_rmem
sysctl net.ipv4.tcp_wmem

# Dropped packets
netstat -i
ip -s link

# Check for packet loss
ping -c 100 8.8.8.8 | tail -1
```

## Network Services
```bash
# What's listening on what port
lsof -i -P -n | grep LISTEN
ss -tlnp

# Find process using specific port
lsof -i :80
ss -tlnp | grep :80
fuser 80/tcp

# Check service status
systemctl status NetworkManager
systemctl status systemd-networkd
systemctl status networking

# Network manager connections
nmcli con show
nmcli dev status
```

## Routing
```bash
# Routing table
ip route show
route -n
netstat -rn

# Default gateway
ip route | grep default
route -n | grep ^0.0.0.0

# Add route (temporary)
ip route add 10.0.0.0/8 via 192.168.1.1

# Check routing to specific IP
ip route get 8.8.8.8
```

## Network Bonding/Teaming
```bash
# Check bonding
cat /proc/net/bonding/bond0 2>/dev/null

# Network team status
teamdctl team0 state 2>/dev/null

# Bridge information
brctl show 2>/dev/null
bridge link show
```

## VPN Diagnostics
```bash
# OpenVPN
systemctl status openvpn*
ps aux | grep openvpn
ip addr show tun0

# WireGuard
wg show
systemctl status wg-quick@*

# IPsec (strongSwan)
ipsec status
systemctl status strongswan
```

## Wireless Diagnostics
```bash
# Wireless interfaces
iwconfig
iw dev

# Wireless networks
nmcli dev wifi list
iwlist scan

# Wireless signal strength
watch -n 1 cat /proc/net/wireless

# Connection info
iw dev wlan0 link
```

## TCP/UDP Tuning
```bash
# Current TCP settings
sysctl -a | grep tcp

# Important TCP parameters
sysctl net.ipv4.tcp_congestion_control
sysctl net.ipv4.tcp_keepalive_time
sysctl net.ipv4.tcp_max_syn_backlog
sysctl net.core.somaxconn

# TCP retransmissions
netstat -s | grep retransmit
ss -ti

# Connection tracking
cat /proc/sys/net/netfilter/nf_conntrack_max
cat /proc/sys/net/netfilter/nf_conntrack_count
conntrack -L 2>/dev/null | wc -l
```

## Network Logs
```bash
# Network-related kernel messages
dmesg | grep -i eth
dmesg | grep -i network
journalctl -k | grep -i network

# NetworkManager logs
journalctl -u NetworkManager -n 100

# DHClient logs
journalctl -u dhclient -n 50
cat /var/log/syslog | grep dhclient | tail -50

# Firewall logs
journalctl -k | grep -i "iptables\|firewall"
grep -i "iptables\|firewall" /var/log/syslog | tail -50
```

## Network Security
```bash
# Check for promiscuous mode (sniffing)
ip link | grep PROMISC
ifconfig | grep PROMISC

# Check for unusual connections
ss -tnp | grep ESTABLISHED
netstat -tnp | grep ESTABLISHED

# SYN flood detection
netstat -an | grep SYN_RECV | wc -l

# Check for port scans in logs
grep "DROP" /var/log/syslog | tail -50
journalctl -k | grep -i "DROP\|REJECT" | tail -50
```

## DNS Security (DNSSEC)
```bash
# Check DNSSEC validation
dig +dnssec google.com
delv google.com  # if bind9-dnsutils installed

# Check if DNSSEC is enabled
cat /etc/systemd/resolved.conf | grep DNSSEC
```

## Troubleshooting Common Issues
```bash
# No internet connection
ping 8.8.8.8  # If fails: routing/hardware issue
ping google.com  # If fails but above works: DNS issue

# Reset network interface
ip link set eth0 down
ip link set eth0 up

# Or using ifconfig
ifdown eth0 && ifup eth0

# Renew DHCP lease
dhclient -r && dhclient  # Release and renew
systemctl restart NetworkManager

# Check for duplicate IPs
arping -D -I eth0 -c 2 $(ip addr show eth0 | grep "inet " | awk '{print $2}' | cut -d/ -f1)

# Test MTU
ping -M do -s 1472 -c 1 google.com  # Should work
ping -M do -s 1500 -c 1 google.com  # May fail if MTU issue
```

## Performance Testing
```bash
# Bandwidth test (iperf)
# Server: iperf -s
# Client: iperf -c <server_ip>

# HTTP download test
wget -O /dev/null http://speedtest.tele2.net/100MB.zip

# Latency test
ping -c 100 8.8.8.8 | tail -3
```

## Output Format
Provide a network diagnostic report with:
- Interface Status
  - All interfaces and their state
  - IP addresses and subnets
- Connectivity Status
  - Internet connectivity (Yes/No)
  - Gateway reachability
  - DNS resolution working
- Active Services
  - Listening ports
  - Established connections count
  - Top connection sources
- Firewall Status
  - Active/Inactive
  - Key rules
- Performance Metrics
  - Packet loss %
  - Latency to key targets
  - Dropped packets
- Issues Found
  - Connection problems
  - DNS failures
  - Firewall blocking
  - Performance issues
- Recommendations
  - Configuration changes
  - Security improvements
  - Performance tuning

# Ubuntu 24.04 LTS VPS Setup with WireGuard and MikroTik Integration - Complete A to Z Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [VPS Initial Setup](#vps-initial-setup)
4. [WireGuard Installation](#wireguard-installation)
5. [WireGuard Configuration](#wireguard-configuration)
6. [MikroTik Router Setup](#mikrotik-router-setup)
7. [Network Architecture](#network-architecture)
8. [Firewall Configuration](#firewall-configuration)
9. [Security Best Practices](#security-best-practices)
10. [Monitoring and Maintenance](#monitoring-and-maintenance)
11. [Troubleshooting](#troubleshooting)

---

## Introduction

This comprehensive guide covers setting up a secure Ubuntu 24.04 LTS VPS with WireGuard VPN and integrating it with MikroTik routers for hotspot management. The setup enables secure remote access, centralized management, and enhanced network security.

### Key Components
- **Ubuntu 24.04 LTS**: Stable server operating system
- **WireGuard**: Modern, high-performance VPN protocol
- **MikroTik**: RouterOS for hotspot and network management
- **skynity/mikhmon**: Web-based hotspot management system

---

## Prerequisites

### Hardware Requirements
- **VPS**: Minimum 1 CPU, 2GB RAM, 20GB SSD
- **MikroTik**: RouterBOARD with RouterOS 6.x or 7.x
- **Network**: Static IP for VPS, dynamic or static IP for MikroTik

### Software Requirements
- Ubuntu 24.04 LTS (Server Edition)
- WireGuard tools
- MikroTik RouterOS
- Web server (Nginx/Apache)

### Network Requirements
- Public static IP for VPS
- Open ports: 22 (SSH), 80 (HTTP), 443 (HTTPS), 51820 (WireGuard)
- DNS records configured

---

## VPS Initial Setup

### 1. Connect to VPS
```bash
# Connect via SSH
ssh root@your-vps-ip

# Update system
apt update && apt upgrade -y

# Set hostname
hostnamectl set-hostname vpn-server
```

### 2. Create Non-Root User
```bash
# Create new user
adduser admin

# Add to sudo group
usermod -aG sudo admin

# Copy SSH keys
mkdir -p /home/admin/.ssh
cp /root/.ssh/authorized_keys /home/admin/.ssh/
chown -R admin:admin /home/admin/.ssh
chmod 700 /home/admin/.ssh
chmod 600 /home/admin/.ssh/authorized_keys
```

### 3. Configure SSH Security
```bash
# Edit SSH config
nano /etc/ssh/sshd_config

# Recommended settings:
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
Port 2222
AllowUsers admin

# Restart SSH
systemctl restart sshd
```

### 4. Configure Timezone and NTP
```bash
# Set timezone
timedatectl set-timezone Asia/Dhaka

# Install NTP
apt install chrony -y
systemctl enable chrony
systemctl start chrony
```

### 5. Enable Automatic Updates
```bash
# Install unattended-upgrades
apt install unattended-upgrades -y

# Configure
dpkg-reconfigure -plow unattended-upgrades
```

---

## WireGuard Installation

### 1. Install WireGuard
```bash
# Add WireGuard repository
apt install software-properties-common -y
add-apt-repository ppa:wireguard/wireguard -y

# Install WireGuard
apt update
apt install wireguard -y

# Install additional tools
apt install wireguard-tools -y
```

### 2. Generate Keys
```bash
# Generate server keys
cd /etc/wireguard
wg genkey | tee server-private.key | wg pubkey > server-public.key

# Generate client keys
wg genkey | tee client1-private.key | wg pubkey > client1-public.key
chmod 600 /etc/wireguard/*.key
```

### 3. Configure WireGuard Server
```bash
# Create server config
nano /etc/wireguard/wg0.conf

[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT
SaveConfig = true

# Start WireGuard
systemctl enable wg-quick@wg0
systemctl start wg-quick@wg0
```

---

## WireGuard Configuration

### 1. MikroTik WireGuard Configuration

#### Enable WireGuard on MikroTik
```bash
# Create WireGuard interface
/interface/wireguard/add name=wg0 listen-port=51820 mtu=1420

# Add peer (VPS)
/interface/wireguard/peers/add interface=wg0 public-key="VPS_PUBLIC_KEY" allowed-address=10.0.0.2/32 persistent-keepalive=25 endpoint-address=VPS_IP endpoint-port=51820
```

#### MikroTik Routing Configuration
```bash
# Add routes
/ip/route/add dst-address=10.0.0.0/24 gateway=wg0 scope=10

# NAT for VPN clients
/ip/firewall/nat/add chain=srcnat out-interface=wg0 src-address=10.0.0.0/24 action=masquerade
```

### 2. VPS Peer Configuration
```bash
# Edit wg0.conf
nano /etc/wireguard/wg0.conf

[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY

[Peer]
# MikroTik
PublicKey = MIKROTIK_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
Endpoint = MIKROTIK_IP:51820
PersistentKeepalive = 25
```

### 3. Add More Peers
```bash
# Generate keys for new peer
wg genkey | tee peer2-private.key | wg pubkey > peer2-public.key

# Add to server config
[Peer]
PublicKey = PEER2_PUBLIC_KEY
AllowedIPs = 10.0.0.3/32

# Client config for peer2
[Interface]
PrivateKey = PEER2_PRIVATE_KEY
Address = 10.0.0.3/24
DNS = 8.8.8.8

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = VPS_IP:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

---

## MikroTik Router Setup

### 1. Basic Router Configuration
```bash
# Set identity
/system/identity/set name=MikroTik-Hotspot

# Configure IP address
/ip/address/add address=192.168.1.1/24 interface=ether1 network=192.168.1.0

# Configure default route
/ip/route/add dst-address=0.0.0.0/0 gateway=192.168.1.254
```

### 2. Hotspot Setup
```bash
# Add hotspot interface
/ip/hotspot/interface/add interface=ether2 disabled=no

# Setup hotspot
/ip/hotspot/setup
```

### 3. Enable API for Remote Management
```bash
# Enable API
/ip/service/enable api

# Set API port
/ip/service/set api port=8728
```

### 4. Connect to WireGuard VPN
```bash
# Add WireGuard peer (already configured in WireGuard section)
/interface/wireguard/peers/add interface=wg0 \
    public-key="VPS_PUBLIC_KEY" \
    allowed-address=10.0.0.2/32 \
    persistent-keepalive=25 \
    endpoint-address=VPS_IP \
    endpoint-port=51820
```

---

## Network Architecture

### Network Diagram
```
Internet
    │
    ├──► VPS (Ubuntu 24.04)
    │       ├── WireGuard VPN (10.0.0.1)
    │       ├── Web Server (skynity/mikhmon)
    │       └── Firewall
    │
    └──► MikroTik Router
            ├── WireGuard (10.0.0.2)
            ├── Hotspot Network (192.168.1.0/24)
            └── Client Network (192.168.2.0/24)
```

### IP Address Scheme
| Component | IP Address | Purpose |
|-----------|-----------|---------|
| VPS VPN | 10.0.0.1 | WireGuard server |
| MikroTik VPN | 10.0.0.2 | WireGuard client |
| MikroTik LAN | 192.168.1.1 | Hotspot gateway |
| Hotspot Clients | 192.168.1.0/24 | User devices |

### Routing Configuration
```bash
# On VPS - Allow forwarding
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

# NAT configuration
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
```

---

## Firewall Configuration

### 1. VPS Firewall (UFW)
```bash
# Install UFW
apt install ufw -y

# Configure rules
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 51820/udp

# Enable firewall
ufw enable

# Check status
ufw status verbose
```

### 2. MikroTik Firewall Rules
```bash
# Allow established/related connections
/ip/firewall/filter/add chain=input connection-state=established,related action=accept

# Allow WireGuard
/ip/firewall/filter/add chain=input protocol=udp dst-port=51820 action=accept

# Allow hotspot traffic
/ip/firewall/filter/add chain=forward connection-state=established,related action=accept

# Drop invalid connections
/ip/firewall/filter/add chain=forward connection-state=invalid action=drop

# Allow HTTP/HTTPS for hotspot
/ip/firewall/filter/add chain=forward dst-port=80 action=accept protocol=tcp
/ip/firewall/filter/add chain=forward dst-port=443 action=accept protocol=tcp
```

### 3. NAT Configuration
```bash
# MASQUERADE for VPN clients
/ip/firewall/nat/add chain=srcnat out-interface=ether1 action=masquerade

# Port forwarding for hotspot (if needed)
/ip/firewall/nat/add chain=dstnat dst-port=80 protocol=tcp action=dst-nat to-addresses=192.168.1.1
```

---

## Security Best Practices

### 1. SSH Hardening
```bash
# Edit sshd_config
nano /etc/ssh/sshd_config

# Add these settings
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
MaxSessions 5
X11Forwarding no
AllowTcpForwarding no
PermitEmptyPasswords no
```

### 2. Fail2Ban Installation
```bash
# Install Fail2Ban
apt install fail2ban -y

# Configure
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
nano /etc/fail2ban/jail.local

# Enable SSH protection
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

# Start service
systemctl enable fail2ban
systemctl start fail2ban
```

### 3. WireGuard Security
```bash
# Use aggressive key rotation
wg genkey | tee /etc/wireguard/private.key
chmod 600 /etc/wireguard/private.key

# Disable persistent keepalive for non-essential peers
# Only enable for necessary connections
```

### 4. Regular Updates
```bash
# Configure automatic security updates
dpkg-reconfigure -plow unattended-upgrades

# Manual update schedule
apt update && apt upgrade -y
```

---

## Monitoring and Maintenance

### 1. WireGuard Monitoring
```bash
# Check VPN status
wg show

# Monitor traffic
watch -n 1 wg show

# Check interface
ip addr show wg0
```

### 2. System Monitoring
```bash
# Install monitoring tools
apt install htop iotop nethogs -y

# Monitor network connections
ss -tuln

# Check firewall status
iptables -L -v -n
```

### 3. Log Monitoring
```bash
# View WireGuard logs
journalctl -u wg-quick@wg0 -f

# Check system logs
tail -f /var/log/syslog

# Monitor authentication
tail -f /var/log/auth.log
```

### 4. Backup Configuration
```bash
# Backup WireGuard config
cp /etc/wireguard/wg0.conf /backup/wg0.conf.$(date +%Y%m%d)

# Backup MikroTik config
/system/sbackup save name=backup-$(date +%Y%m%d)
```

---

## Troubleshooting

### 1. WireGuard Connection Issues

#### Problem: Connection Timeout
```bash
# Check if WireGuard is running
systemctl status wg-quick@wg0

# Verify port is listening
ss -uln | grep 51820

# Check firewall rules
ufw status
iptables -L INPUT -v -n | grep 51820
```

#### Problem: Handshake Failed
```bash
# Verify keys match
wg pubkey < /etc/wireguard/client-private.key

# Check endpoint configuration
wg show

# Verify routing
ip route show
```

### 2. MikroTik Connection Issues

#### Problem: Cannot Connect to MikroTik
```bash
# Test connectivity
ping 192.168.1.1

# Check API service
/tool/fetch url="http://192.168.1.1/rest/interface" host=192.168.1.1

# Verify VPN tunnel
/interface/wireguard/print
/interface/wireguard/peers/print
```

### 3. Network Issues

#### Problem: No Internet Access
```bash
# Check IP forwarding
sysctl net.ipv4.ip_forward

# Verify NAT rules
iptables -t nat -L POSTROUTING -v -n

# Check routing table
ip route show
```

### 4. Performance Issues

#### Problem: Slow VPN Performance
```bash
# Check CPU usage
htop

# Monitor network speed
speedtest-cli

# Check for packet loss
mtr -r -c 10 VPS_IP
```

---

## Additional Resources

### Useful Commands
```bash
# WireGuard
wg show
wg-quick down wg0
wg-quick up wg0

# MikroTik
/tool/fetch url="http://192.168.1.1/rest/interface" user=admin password=your_pass
/system/reboot

# System
reboot
shutdown -h now
```

### Security Checklist
- [ ] Change default passwords
- [ ] Disable root SSH login
- [ ] Enable firewall
- [ ] Configure fail2ban
- [ ] Set up regular backups
- [ ] Monitor logs regularly
- [ ] Update system frequently
- [ ] Use strong keys
- [ ] Enable 2FA where possible
- [ ] Document all configurations

### Performance Optimization
- [ ] Enable BBR congestion control
- [ ] Optimize buffer sizes
- [ ] Configure QoS if needed
- [ ] Monitor bandwidth usage
- [ ] Regular maintenance checks

---

## Conclusion

This guide provides a complete setup for Ubuntu 24.04 LTS VPS with WireGuard and MikroTik integration. Following these steps will create a secure, performant, and manageable network infrastructure.

### Next Steps
1. Test all configurations
2. Document specific settings
3. Set up monitoring alerts
4. Create runbooks for common issues
5. Schedule regular maintenance

### Support Resources
- WireGuard Official Documentation
- MikroTik Wiki
- Ubuntu Server Guide
- Community Forums

---

**Last Updated**: 2026-02-08
**Version**: 1.0

# n8n Installation Guide

n8n is a free and open-source workflow automation tool. n8n provides visual workflow automation with 200+ integrations, serving as a self-hosted alternative to Zapier, IFTTT, or Microsoft Power Automate

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores recommended
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for workflows
  - Network: HTTPS for webhooks
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5678 (default n8n port)
  - Webhook endpoints
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install n8n
sudo dnf install -y n8n

# Enable and start service
sudo systemctl enable --now n8n

# Configure firewall
sudo firewall-cmd --permanent --add-port=5678/tcp
sudo firewall-cmd --reload

# Verify installation
n8n --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install n8n
sudo apt install -y n8n

# Enable and start service
sudo systemctl enable --now n8n

# Configure firewall
sudo ufw allow 5678

# Verify installation
n8n --version
```

### Arch Linux

```bash
# Install n8n
sudo pacman -S n8n

# Enable and start service
sudo systemctl enable --now n8n

# Verify installation
n8n --version
```

### Alpine Linux

```bash
# Install n8n
apk add --no-cache n8n

# Enable and start service
rc-update add n8n default
rc-service n8n start

# Verify installation
n8n --version
```

### openSUSE/SLES

```bash
# Install n8n
sudo zypper install -y n8n

# Enable and start service
sudo systemctl enable --now n8n

# Configure firewall
sudo firewall-cmd --permanent --add-port=5678/tcp
sudo firewall-cmd --reload

# Verify installation
n8n --version
```

### macOS

```bash
# Using Homebrew
brew install n8n

# Start service
brew services start n8n

# Verify installation
n8n --version
```

### FreeBSD

```bash
# Using pkg
pkg install n8n

# Enable in rc.conf
echo 'n8n_enable="YES"' >> /etc/rc.conf

# Start service
service n8n start

# Verify installation
n8n --version
```

### Windows

```bash
# Using Chocolatey
choco install n8n

# Or using Scoop
scoop install n8n

# Verify installation
n8n --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/n8n

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
n8n --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable n8n

# Start service
sudo systemctl start n8n

# Stop service
sudo systemctl stop n8n

# Restart service
sudo systemctl restart n8n

# Check status
sudo systemctl status n8n

# View logs
sudo journalctl -u n8n -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add n8n default

# Start service
rc-service n8n start

# Stop service
rc-service n8n stop

# Restart service
rc-service n8n restart

# Check status
rc-service n8n status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'n8n_enable="YES"' >> /etc/rc.conf

# Start service
service n8n start

# Stop service
service n8n stop

# Restart service
service n8n restart

# Check status
service n8n status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start n8n
brew services stop n8n
brew services restart n8n

# Check status
brew services list | grep n8n
```

### Windows Service Manager

```powershell
# Start service
net start n8n

# Stop service
net stop n8n

# Using PowerShell
Start-Service n8n
Stop-Service n8n
Restart-Service n8n

# Check status
Get-Service n8n
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream n8n_backend {
    server 127.0.0.1:5678;
}

server {
    listen 80;
    server_name n8n.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name n8n.example.com;

    ssl_certificate /etc/ssl/certs/n8n.example.com.crt;
    ssl_certificate_key /etc/ssl/private/n8n.example.com.key;

    location / {
        proxy_pass http://n8n_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName n8n.example.com
    Redirect permanent / https://n8n.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName n8n.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/n8n.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/n8n.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5678/
    ProxyPassReverse / http://127.0.0.1:5678/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend n8n_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/n8n.pem
    redirect scheme https if !{ ssl_fc }
    default_backend n8n_backend

backend n8n_backend
    balance roundrobin
    server n8n1 127.0.0.1:5678 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R n8n:n8n /etc/n8n
sudo chmod 750 /etc/n8n

# Configure firewall
sudo firewall-cmd --permanent --add-port=5678/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status n8n

# View logs
sudo journalctl -u n8n -f

# Monitor resource usage
top -p $(pgrep n8n)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/n8n"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/n8n-backup-$DATE.tar.gz" /etc/n8n /var/lib/n8n

echo "Backup completed: $BACKUP_DIR/n8n-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop n8n

# Restore from backup
tar -xzf /backup/n8n/n8n-backup-*.tar.gz -C /

# Start service
sudo systemctl start n8n
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u n8n -n 100
sudo tail -f /var/log/n8n/n8n.log

# Check configuration
n8n --version

# Check permissions
ls -la /etc/n8n
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5678

# Test connectivity
telnet localhost 5678

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep n8n)

# Check disk I/O
iotop -p $(pgrep n8n)

# Check connections
ss -an | grep 5678
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  n8n:
    image: n8n:latest
    ports:
      - "5678:5678"
    volumes:
      - ./config:/etc/n8n
      - ./data:/var/lib/n8n
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update n8n

# Debian/Ubuntu
sudo apt update && sudo apt upgrade n8n

# Arch Linux
sudo pacman -Syu n8n

# Alpine Linux
apk update && apk upgrade n8n

# openSUSE
sudo zypper update n8n

# FreeBSD
pkg update && pkg upgrade n8n

# Always backup before updates
tar -czf /backup/n8n-pre-update-$(date +%Y%m%d).tar.gz /etc/n8n

# Restart after updates
sudo systemctl restart n8n
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/n8n

# Clean old logs
find /var/log/n8n -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/n8n
```

## Additional Resources

- Official Documentation: https://docs.n8n.org/
- GitHub Repository: https://github.com/n8n/n8n
- Community Forum: https://forum.n8n.org/
- Best Practices Guide: https://docs.n8n.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

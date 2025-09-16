# gocd Installation Guide

gocd is a free and open-source continuous delivery. ThoughtWorks GoCD provides continuous delivery server

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
  - CPU: 2+ cores
  - RAM: 4GB minimum
  - Storage: 20GB for artifacts
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8153 (default gocd port)
  - Agent on 8154
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

# Install gocd
sudo dnf install -y gocd

# Enable and start service
sudo systemctl enable --now gocd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8153/tcp
sudo firewall-cmd --reload

# Verify installation
gocd --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gocd
sudo apt install -y gocd

# Enable and start service
sudo systemctl enable --now gocd

# Configure firewall
sudo ufw allow 8153

# Verify installation
gocd --version
```

### Arch Linux

```bash
# Install gocd
sudo pacman -S gocd

# Enable and start service
sudo systemctl enable --now gocd

# Verify installation
gocd --version
```

### Alpine Linux

```bash
# Install gocd
apk add --no-cache gocd

# Enable and start service
rc-update add gocd default
rc-service gocd start

# Verify installation
gocd --version
```

### openSUSE/SLES

```bash
# Install gocd
sudo zypper install -y gocd

# Enable and start service
sudo systemctl enable --now gocd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8153/tcp
sudo firewall-cmd --reload

# Verify installation
gocd --version
```

### macOS

```bash
# Using Homebrew
brew install gocd

# Start service
brew services start gocd

# Verify installation
gocd --version
```

### FreeBSD

```bash
# Using pkg
pkg install gocd

# Enable in rc.conf
echo 'gocd_enable="YES"' >> /etc/rc.conf

# Start service
service gocd start

# Verify installation
gocd --version
```

### Windows

```bash
# Using Chocolatey
choco install gocd

# Or using Scoop
scoop install gocd

# Verify installation
gocd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gocd

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gocd --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gocd

# Start service
sudo systemctl start gocd

# Stop service
sudo systemctl stop gocd

# Restart service
sudo systemctl restart gocd

# Check status
sudo systemctl status gocd

# View logs
sudo journalctl -u gocd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gocd default

# Start service
rc-service gocd start

# Stop service
rc-service gocd stop

# Restart service
rc-service gocd restart

# Check status
rc-service gocd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gocd_enable="YES"' >> /etc/rc.conf

# Start service
service gocd start

# Stop service
service gocd stop

# Restart service
service gocd restart

# Check status
service gocd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gocd
brew services stop gocd
brew services restart gocd

# Check status
brew services list | grep gocd
```

### Windows Service Manager

```powershell
# Start service
net start gocd

# Stop service
net stop gocd

# Using PowerShell
Start-Service gocd
Stop-Service gocd
Restart-Service gocd

# Check status
Get-Service gocd
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gocd_backend {
    server 127.0.0.1:8153;
}

server {
    listen 80;
    server_name gocd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gocd.example.com;

    ssl_certificate /etc/ssl/certs/gocd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gocd.example.com.key;

    location / {
        proxy_pass http://gocd_backend;
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
    ServerName gocd.example.com
    Redirect permanent / https://gocd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gocd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gocd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gocd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8153/
    ProxyPassReverse / http://127.0.0.1:8153/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gocd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gocd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gocd_backend

backend gocd_backend
    balance roundrobin
    server gocd1 127.0.0.1:8153 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gocd:gocd /etc/gocd
sudo chmod 750 /etc/gocd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8153/tcp
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
sudo systemctl status gocd

# View logs
sudo journalctl -u gocd -f

# Monitor resource usage
top -p $(pgrep gocd)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gocd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gocd-backup-$DATE.tar.gz" /etc/gocd /var/lib/gocd

echo "Backup completed: $BACKUP_DIR/gocd-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gocd

# Restore from backup
tar -xzf /backup/gocd/gocd-backup-*.tar.gz -C /

# Start service
sudo systemctl start gocd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gocd -n 100
sudo tail -f /var/log/gocd/gocd.log

# Check configuration
gocd --version

# Check permissions
ls -la /etc/gocd
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8153

# Test connectivity
telnet localhost 8153

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gocd)

# Check disk I/O
iotop -p $(pgrep gocd)

# Check connections
ss -an | grep 8153
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gocd:
    image: gocd:latest
    ports:
      - "8153:8153"
    volumes:
      - ./config:/etc/gocd
      - ./data:/var/lib/gocd
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gocd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gocd

# Arch Linux
sudo pacman -Syu gocd

# Alpine Linux
apk update && apk upgrade gocd

# openSUSE
sudo zypper update gocd

# FreeBSD
pkg update && pkg upgrade gocd

# Always backup before updates
tar -czf /backup/gocd-pre-update-$(date +%Y%m%d).tar.gz /etc/gocd

# Restart after updates
sudo systemctl restart gocd
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gocd

# Clean old logs
find /var/log/gocd -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gocd
```

## Additional Resources

- Official Documentation: https://docs.gocd.org/
- GitHub Repository: https://github.com/gocd/gocd
- Community Forum: https://forum.gocd.org/
- Best Practices Guide: https://docs.gocd.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

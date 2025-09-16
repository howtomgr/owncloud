# owncloud Installation Guide

owncloud is a free and open-source self-hosted file sync and share solution. ownCloud enables file access, sync and sharing capabilities, providing an open-source alternative to commercial cloud storage services

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
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 10GB+ for data
  - Network: HTTPS for web access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default owncloud port)
  - WebDAV support
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

# Install owncloud
sudo dnf install -y owncloud

# Enable and start service
sudo systemctl enable --now owncloud

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
occ --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install owncloud
sudo apt install -y owncloud

# Enable and start service
sudo systemctl enable --now owncloud

# Configure firewall
sudo ufw allow 80/443

# Verify installation
occ --version
```

### Arch Linux

```bash
# Install owncloud
sudo pacman -S owncloud

# Enable and start service
sudo systemctl enable --now owncloud

# Verify installation
occ --version
```

### Alpine Linux

```bash
# Install owncloud
apk add --no-cache owncloud

# Enable and start service
rc-update add owncloud default
rc-service owncloud start

# Verify installation
occ --version
```

### openSUSE/SLES

```bash
# Install owncloud
sudo zypper install -y owncloud

# Enable and start service
sudo systemctl enable --now owncloud

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
occ --version
```

### macOS

```bash
# Using Homebrew
brew install owncloud

# Start service
brew services start owncloud

# Verify installation
occ --version
```

### FreeBSD

```bash
# Using pkg
pkg install owncloud

# Enable in rc.conf
echo 'owncloud_enable="YES"' >> /etc/rc.conf

# Start service
service owncloud start

# Verify installation
occ --version
```

### Windows

```bash
# Using Chocolatey
choco install owncloud

# Or using Scoop
scoop install owncloud

# Verify installation
occ --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/owncloud

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
occ --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable owncloud

# Start service
sudo systemctl start owncloud

# Stop service
sudo systemctl stop owncloud

# Restart service
sudo systemctl restart owncloud

# Check status
sudo systemctl status owncloud

# View logs
sudo journalctl -u owncloud -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add owncloud default

# Start service
rc-service owncloud start

# Stop service
rc-service owncloud stop

# Restart service
rc-service owncloud restart

# Check status
rc-service owncloud status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'owncloud_enable="YES"' >> /etc/rc.conf

# Start service
service owncloud start

# Stop service
service owncloud stop

# Restart service
service owncloud restart

# Check status
service owncloud status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start owncloud
brew services stop owncloud
brew services restart owncloud

# Check status
brew services list | grep owncloud
```

### Windows Service Manager

```powershell
# Start service
net start owncloud

# Stop service
net stop owncloud

# Using PowerShell
Start-Service owncloud
Stop-Service owncloud
Restart-Service owncloud

# Check status
Get-Service owncloud
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream owncloud_backend {
    server 127.0.0.1:80/443;
}

server {
    listen 80;
    server_name owncloud.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name owncloud.example.com;

    ssl_certificate /etc/ssl/certs/owncloud.example.com.crt;
    ssl_certificate_key /etc/ssl/private/owncloud.example.com.key;

    location / {
        proxy_pass http://owncloud_backend;
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
    ServerName owncloud.example.com
    Redirect permanent / https://owncloud.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName owncloud.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/owncloud.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/owncloud.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend owncloud_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/owncloud.pem
    redirect scheme https if !{ ssl_fc }
    default_backend owncloud_backend

backend owncloud_backend
    balance roundrobin
    server owncloud1 127.0.0.1:80/443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R owncloud:owncloud /etc/owncloud
sudo chmod 750 /etc/owncloud

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
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
sudo systemctl status owncloud

# View logs
sudo journalctl -u owncloud -f

# Monitor resource usage
top -p $(pgrep owncloud)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/owncloud"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/owncloud-backup-$DATE.tar.gz" /etc/owncloud /var/lib/owncloud

echo "Backup completed: $BACKUP_DIR/owncloud-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop owncloud

# Restore from backup
tar -xzf /backup/owncloud/owncloud-backup-*.tar.gz -C /

# Start service
sudo systemctl start owncloud
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u owncloud -n 100
sudo tail -f /var/log/owncloud/owncloud.log

# Check configuration
occ --version

# Check permissions
ls -la /etc/owncloud
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443

# Test connectivity
telnet localhost 80/443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep owncloud)

# Check disk I/O
iotop -p $(pgrep owncloud)

# Check connections
ss -an | grep 80/443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  owncloud:
    image: owncloud:latest
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/etc/owncloud
      - ./data:/var/lib/owncloud
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update owncloud

# Debian/Ubuntu
sudo apt update && sudo apt upgrade owncloud

# Arch Linux
sudo pacman -Syu owncloud

# Alpine Linux
apk update && apk upgrade owncloud

# openSUSE
sudo zypper update owncloud

# FreeBSD
pkg update && pkg upgrade owncloud

# Always backup before updates
tar -czf /backup/owncloud-pre-update-$(date +%Y%m%d).tar.gz /etc/owncloud

# Restart after updates
sudo systemctl restart owncloud
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/owncloud

# Clean old logs
find /var/log/owncloud -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/owncloud
```

## Additional Resources

- Official Documentation: https://docs.owncloud.org/
- GitHub Repository: https://github.com/owncloud/owncloud
- Community Forum: https://forum.owncloud.org/
- Best Practices Guide: https://docs.owncloud.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

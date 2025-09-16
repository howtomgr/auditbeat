# auditbeat Installation Guide

auditbeat is a free and open-source audit data shipper. Auditbeat ships audit framework data and monitors integrity

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
  - CPU: 1 core minimum
  - RAM: 256MB minimum
  - Storage: 1GB for data
  - Network: System audit
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5066 (default auditbeat port)
  - None
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

# Install auditbeat
sudo dnf install -y auditbeat

# Enable and start service
sudo systemctl enable --now auditbeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
sudo firewall-cmd --reload

# Verify installation
auditbeat --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install auditbeat
sudo apt install -y auditbeat

# Enable and start service
sudo systemctl enable --now auditbeat

# Configure firewall
sudo ufw allow 5066

# Verify installation
auditbeat --version
```

### Arch Linux

```bash
# Install auditbeat
sudo pacman -S auditbeat

# Enable and start service
sudo systemctl enable --now auditbeat

# Verify installation
auditbeat --version
```

### Alpine Linux

```bash
# Install auditbeat
apk add --no-cache auditbeat

# Enable and start service
rc-update add auditbeat default
rc-service auditbeat start

# Verify installation
auditbeat --version
```

### openSUSE/SLES

```bash
# Install auditbeat
sudo zypper install -y auditbeat

# Enable and start service
sudo systemctl enable --now auditbeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
sudo firewall-cmd --reload

# Verify installation
auditbeat --version
```

### macOS

```bash
# Using Homebrew
brew install auditbeat

# Start service
brew services start auditbeat

# Verify installation
auditbeat --version
```

### FreeBSD

```bash
# Using pkg
pkg install auditbeat

# Enable in rc.conf
echo 'auditbeat_enable="YES"' >> /etc/rc.conf

# Start service
service auditbeat start

# Verify installation
auditbeat --version
```

### Windows

```bash
# Using Chocolatey
choco install auditbeat

# Or using Scoop
scoop install auditbeat

# Verify installation
auditbeat --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/auditbeat

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
auditbeat --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable auditbeat

# Start service
sudo systemctl start auditbeat

# Stop service
sudo systemctl stop auditbeat

# Restart service
sudo systemctl restart auditbeat

# Check status
sudo systemctl status auditbeat

# View logs
sudo journalctl -u auditbeat -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add auditbeat default

# Start service
rc-service auditbeat start

# Stop service
rc-service auditbeat stop

# Restart service
rc-service auditbeat restart

# Check status
rc-service auditbeat status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'auditbeat_enable="YES"' >> /etc/rc.conf

# Start service
service auditbeat start

# Stop service
service auditbeat stop

# Restart service
service auditbeat restart

# Check status
service auditbeat status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start auditbeat
brew services stop auditbeat
brew services restart auditbeat

# Check status
brew services list | grep auditbeat
```

### Windows Service Manager

```powershell
# Start service
net start auditbeat

# Stop service
net stop auditbeat

# Using PowerShell
Start-Service auditbeat
Stop-Service auditbeat
Restart-Service auditbeat

# Check status
Get-Service auditbeat
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream auditbeat_backend {
    server 127.0.0.1:5066;
}

server {
    listen 80;
    server_name auditbeat.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name auditbeat.example.com;

    ssl_certificate /etc/ssl/certs/auditbeat.example.com.crt;
    ssl_certificate_key /etc/ssl/private/auditbeat.example.com.key;

    location / {
        proxy_pass http://auditbeat_backend;
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
    ServerName auditbeat.example.com
    Redirect permanent / https://auditbeat.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName auditbeat.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/auditbeat.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/auditbeat.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5066/
    ProxyPassReverse / http://127.0.0.1:5066/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend auditbeat_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/auditbeat.pem
    redirect scheme https if !{ ssl_fc }
    default_backend auditbeat_backend

backend auditbeat_backend
    balance roundrobin
    server auditbeat1 127.0.0.1:5066 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R auditbeat:auditbeat /etc/auditbeat
sudo chmod 750 /etc/auditbeat

# Configure firewall
sudo firewall-cmd --permanent --add-port=5066/tcp
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
sudo systemctl status auditbeat

# View logs
sudo journalctl -u auditbeat -f

# Monitor resource usage
top -p $(pgrep auditbeat)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/auditbeat"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/auditbeat-backup-$DATE.tar.gz" /etc/auditbeat /var/lib/auditbeat

echo "Backup completed: $BACKUP_DIR/auditbeat-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop auditbeat

# Restore from backup
tar -xzf /backup/auditbeat/auditbeat-backup-*.tar.gz -C /

# Start service
sudo systemctl start auditbeat
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u auditbeat -n 100
sudo tail -f /var/log/auditbeat/auditbeat.log

# Check configuration
auditbeat --version

# Check permissions
ls -la /etc/auditbeat
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5066

# Test connectivity
telnet localhost 5066

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep auditbeat)

# Check disk I/O
iotop -p $(pgrep auditbeat)

# Check connections
ss -an | grep 5066
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  auditbeat:
    image: auditbeat:latest
    ports:
      - "5066:5066"
    volumes:
      - ./config:/etc/auditbeat
      - ./data:/var/lib/auditbeat
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update auditbeat

# Debian/Ubuntu
sudo apt update && sudo apt upgrade auditbeat

# Arch Linux
sudo pacman -Syu auditbeat

# Alpine Linux
apk update && apk upgrade auditbeat

# openSUSE
sudo zypper update auditbeat

# FreeBSD
pkg update && pkg upgrade auditbeat

# Always backup before updates
tar -czf /backup/auditbeat-pre-update-$(date +%Y%m%d).tar.gz /etc/auditbeat

# Restart after updates
sudo systemctl restart auditbeat
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/auditbeat

# Clean old logs
find /var/log/auditbeat -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/auditbeat
```

## Additional Resources

- Official Documentation: https://docs.auditbeat.org/
- GitHub Repository: https://github.com/auditbeat/auditbeat
- Community Forum: https://forum.auditbeat.org/
- Best Practices Guide: https://docs.auditbeat.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

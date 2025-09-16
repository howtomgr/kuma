# kuma Installation Guide

kuma is a free and open-source service mesh. Kuma provides universal service mesh for multi-zone deployments

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
  - RAM: 2GB minimum
  - Storage: 5GB for config
  - Network: Various protocols
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5681 (default kuma port)
  - GUI on 5683
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

# Install kuma
sudo dnf install -y kuma

# Enable and start service
sudo systemctl enable --now kuma

# Configure firewall
sudo firewall-cmd --permanent --add-port=5681/tcp
sudo firewall-cmd --reload

# Verify installation
kuma --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install kuma
sudo apt install -y kuma

# Enable and start service
sudo systemctl enable --now kuma

# Configure firewall
sudo ufw allow 5681

# Verify installation
kuma --version
```

### Arch Linux

```bash
# Install kuma
sudo pacman -S kuma

# Enable and start service
sudo systemctl enable --now kuma

# Verify installation
kuma --version
```

### Alpine Linux

```bash
# Install kuma
apk add --no-cache kuma

# Enable and start service
rc-update add kuma default
rc-service kuma start

# Verify installation
kuma --version
```

### openSUSE/SLES

```bash
# Install kuma
sudo zypper install -y kuma

# Enable and start service
sudo systemctl enable --now kuma

# Configure firewall
sudo firewall-cmd --permanent --add-port=5681/tcp
sudo firewall-cmd --reload

# Verify installation
kuma --version
```

### macOS

```bash
# Using Homebrew
brew install kuma

# Start service
brew services start kuma

# Verify installation
kuma --version
```

### FreeBSD

```bash
# Using pkg
pkg install kuma

# Enable in rc.conf
echo 'kuma_enable="YES"' >> /etc/rc.conf

# Start service
service kuma start

# Verify installation
kuma --version
```

### Windows

```bash
# Using Chocolatey
choco install kuma

# Or using Scoop
scoop install kuma

# Verify installation
kuma --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/kuma

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
kuma --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable kuma

# Start service
sudo systemctl start kuma

# Stop service
sudo systemctl stop kuma

# Restart service
sudo systemctl restart kuma

# Check status
sudo systemctl status kuma

# View logs
sudo journalctl -u kuma -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add kuma default

# Start service
rc-service kuma start

# Stop service
rc-service kuma stop

# Restart service
rc-service kuma restart

# Check status
rc-service kuma status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'kuma_enable="YES"' >> /etc/rc.conf

# Start service
service kuma start

# Stop service
service kuma stop

# Restart service
service kuma restart

# Check status
service kuma status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start kuma
brew services stop kuma
brew services restart kuma

# Check status
brew services list | grep kuma
```

### Windows Service Manager

```powershell
# Start service
net start kuma

# Stop service
net stop kuma

# Using PowerShell
Start-Service kuma
Stop-Service kuma
Restart-Service kuma

# Check status
Get-Service kuma
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream kuma_backend {
    server 127.0.0.1:5681;
}

server {
    listen 80;
    server_name kuma.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name kuma.example.com;

    ssl_certificate /etc/ssl/certs/kuma.example.com.crt;
    ssl_certificate_key /etc/ssl/private/kuma.example.com.key;

    location / {
        proxy_pass http://kuma_backend;
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
    ServerName kuma.example.com
    Redirect permanent / https://kuma.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName kuma.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/kuma.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/kuma.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5681/
    ProxyPassReverse / http://127.0.0.1:5681/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend kuma_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/kuma.pem
    redirect scheme https if !{ ssl_fc }
    default_backend kuma_backend

backend kuma_backend
    balance roundrobin
    server kuma1 127.0.0.1:5681 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R kuma:kuma /etc/kuma
sudo chmod 750 /etc/kuma

# Configure firewall
sudo firewall-cmd --permanent --add-port=5681/tcp
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
sudo systemctl status kuma

# View logs
sudo journalctl -u kuma -f

# Monitor resource usage
top -p $(pgrep kuma)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/kuma"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/kuma-backup-$DATE.tar.gz" /etc/kuma /var/lib/kuma

echo "Backup completed: $BACKUP_DIR/kuma-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop kuma

# Restore from backup
tar -xzf /backup/kuma/kuma-backup-*.tar.gz -C /

# Start service
sudo systemctl start kuma
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u kuma -n 100
sudo tail -f /var/log/kuma/kuma.log

# Check configuration
kuma --version

# Check permissions
ls -la /etc/kuma
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5681

# Test connectivity
telnet localhost 5681

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep kuma)

# Check disk I/O
iotop -p $(pgrep kuma)

# Check connections
ss -an | grep 5681
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  kuma:
    image: kuma:latest
    ports:
      - "5681:5681"
    volumes:
      - ./config:/etc/kuma
      - ./data:/var/lib/kuma
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update kuma

# Debian/Ubuntu
sudo apt update && sudo apt upgrade kuma

# Arch Linux
sudo pacman -Syu kuma

# Alpine Linux
apk update && apk upgrade kuma

# openSUSE
sudo zypper update kuma

# FreeBSD
pkg update && pkg upgrade kuma

# Always backup before updates
tar -czf /backup/kuma-pre-update-$(date +%Y%m%d).tar.gz /etc/kuma

# Restart after updates
sudo systemctl restart kuma
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/kuma

# Clean old logs
find /var/log/kuma -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/kuma
```

## Additional Resources

- Official Documentation: https://docs.kuma.org/
- GitHub Repository: https://github.com/kuma/kuma
- Community Forum: https://forum.kuma.org/
- Best Practices Guide: https://docs.kuma.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

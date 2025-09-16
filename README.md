# registry Installation Guide

registry is a free and open-source Docker image registry. Docker Registry stores and distributes Docker images, providing a self-hosted alternative to Docker Hub or cloud registries

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
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 10GB+ for images
  - Network: HTTPS recommended
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5000 (default registry port)
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

# Install registry
sudo dnf install -y docker-registry

# Enable and start service
sudo systemctl enable --now docker-registry

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload

# Verify installation
registry --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install registry
sudo apt install -y docker-registry

# Enable and start service
sudo systemctl enable --now docker-registry

# Configure firewall
sudo ufw allow 5000

# Verify installation
registry --version
```

### Arch Linux

```bash
# Install registry
sudo pacman -S docker-registry

# Enable and start service
sudo systemctl enable --now docker-registry

# Verify installation
registry --version
```

### Alpine Linux

```bash
# Install registry
apk add --no-cache docker-registry

# Enable and start service
rc-update add docker-registry default
rc-service docker-registry start

# Verify installation
registry --version
```

### openSUSE/SLES

```bash
# Install registry
sudo zypper install -y docker-registry

# Enable and start service
sudo systemctl enable --now docker-registry

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload

# Verify installation
registry --version
```

### macOS

```bash
# Using Homebrew
brew install docker-registry

# Start service
brew services start docker-registry

# Verify installation
registry --version
```

### FreeBSD

```bash
# Using pkg
pkg install docker-registry

# Enable in rc.conf
echo 'docker-registry_enable="YES"' >> /etc/rc.conf

# Start service
service docker-registry start

# Verify installation
registry --version
```

### Windows

```bash
# Using Chocolatey
choco install docker-registry

# Or using Scoop
scoop install docker-registry

# Verify installation
registry --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/docker-registry

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
registry --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable docker-registry

# Start service
sudo systemctl start docker-registry

# Stop service
sudo systemctl stop docker-registry

# Restart service
sudo systemctl restart docker-registry

# Check status
sudo systemctl status docker-registry

# View logs
sudo journalctl -u docker-registry -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add docker-registry default

# Start service
rc-service docker-registry start

# Stop service
rc-service docker-registry stop

# Restart service
rc-service docker-registry restart

# Check status
rc-service docker-registry status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'docker-registry_enable="YES"' >> /etc/rc.conf

# Start service
service docker-registry start

# Stop service
service docker-registry stop

# Restart service
service docker-registry restart

# Check status
service docker-registry status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start docker-registry
brew services stop docker-registry
brew services restart docker-registry

# Check status
brew services list | grep docker-registry
```

### Windows Service Manager

```powershell
# Start service
net start docker-registry

# Stop service
net stop docker-registry

# Using PowerShell
Start-Service docker-registry
Stop-Service docker-registry
Restart-Service docker-registry

# Check status
Get-Service docker-registry
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream docker-registry_backend {
    server 127.0.0.1:5000;
}

server {
    listen 80;
    server_name docker-registry.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name docker-registry.example.com;

    ssl_certificate /etc/ssl/certs/docker-registry.example.com.crt;
    ssl_certificate_key /etc/ssl/private/docker-registry.example.com.key;

    location / {
        proxy_pass http://docker-registry_backend;
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
    ServerName docker-registry.example.com
    Redirect permanent / https://docker-registry.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName docker-registry.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/docker-registry.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/docker-registry.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5000/
    ProxyPassReverse / http://127.0.0.1:5000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend docker-registry_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/docker-registry.pem
    redirect scheme https if !{ ssl_fc }
    default_backend docker-registry_backend

backend docker-registry_backend
    balance roundrobin
    server docker-registry1 127.0.0.1:5000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R docker-registry:docker-registry /etc/docker-registry
sudo chmod 750 /etc/docker-registry

# Configure firewall
sudo firewall-cmd --permanent --add-port=5000/tcp
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
sudo systemctl status docker-registry

# View logs
sudo journalctl -u docker-registry -f

# Monitor resource usage
top -p $(pgrep docker-registry)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/docker-registry"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/docker-registry-backup-$DATE.tar.gz" /etc/docker-registry /var/lib/docker-registry

echo "Backup completed: $BACKUP_DIR/docker-registry-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop docker-registry

# Restore from backup
tar -xzf /backup/docker-registry/docker-registry-backup-*.tar.gz -C /

# Start service
sudo systemctl start docker-registry
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u docker-registry -n 100
sudo tail -f /var/log/docker-registry/docker-registry.log

# Check configuration
registry --version

# Check permissions
ls -la /etc/docker-registry
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5000

# Test connectivity
telnet localhost 5000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep docker-registry)

# Check disk I/O
iotop -p $(pgrep docker-registry)

# Check connections
ss -an | grep 5000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  docker-registry:
    image: docker-registry:latest
    ports:
      - "5000:5000"
    volumes:
      - ./config:/etc/docker-registry
      - ./data:/var/lib/docker-registry
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update docker-registry

# Debian/Ubuntu
sudo apt update && sudo apt upgrade docker-registry

# Arch Linux
sudo pacman -Syu docker-registry

# Alpine Linux
apk update && apk upgrade docker-registry

# openSUSE
sudo zypper update docker-registry

# FreeBSD
pkg update && pkg upgrade docker-registry

# Always backup before updates
tar -czf /backup/docker-registry-pre-update-$(date +%Y%m%d).tar.gz /etc/docker-registry

# Restart after updates
sudo systemctl restart docker-registry
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/docker-registry

# Clean old logs
find /var/log/docker-registry -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/docker-registry
```

## Additional Resources

- Official Documentation: https://docs.docker-registry.org/
- GitHub Repository: https://github.com/docker-registry/docker-registry
- Community Forum: https://forum.docker-registry.org/
- Best Practices Guide: https://docs.docker-registry.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

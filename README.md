# vault Installation Guide

vault is a free and open-source secrets management and data protection platform. Developed by HashiCorp, Vault provides enterprise-grade secrets management, encryption as a service, and privileged access management, serving as an open-source alternative to AWS Secrets Manager, Azure Key Vault, or CyberArk

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
  - RAM: 1GB minimum (2GB+ for production)
  - Storage: 10GB for installation and data
  - Network: HTTPS connectivity for clients
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8200 (default vault port)
  - Port 8201 for cluster communication
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

# Install vault
sudo dnf install -y vault

# Enable and start service
sudo systemctl enable --now vault

# Configure firewall
sudo firewall-cmd --permanent --add-port=8200/tcp
sudo firewall-cmd --reload

# Verify installation
vault --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install vault
sudo apt install -y vault

# Enable and start service
sudo systemctl enable --now vault

# Configure firewall
sudo ufw allow 8200

# Verify installation
vault --version
```

### Arch Linux

```bash
# Install vault
sudo pacman -S vault

# Enable and start service
sudo systemctl enable --now vault

# Verify installation
vault --version
```

### Alpine Linux

```bash
# Install vault
apk add --no-cache vault

# Enable and start service
rc-update add vault default
rc-service vault start

# Verify installation
vault --version
```

### openSUSE/SLES

```bash
# Install vault
sudo zypper install -y vault

# Enable and start service
sudo systemctl enable --now vault

# Configure firewall
sudo firewall-cmd --permanent --add-port=8200/tcp
sudo firewall-cmd --reload

# Verify installation
vault --version
```

### macOS

```bash
# Using Homebrew
brew install vault

# Start service
brew services start vault

# Verify installation
vault --version
```

### FreeBSD

```bash
# Using pkg
pkg install vault

# Enable in rc.conf
echo 'vault_enable="YES"' >> /etc/rc.conf

# Start service
service vault start

# Verify installation
vault --version
```

### Windows

```bash
# Using Chocolatey
choco install vault

# Or using Scoop
scoop install vault

# Verify installation
vault --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/vault

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
vault --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable vault

# Start service
sudo systemctl start vault

# Stop service
sudo systemctl stop vault

# Restart service
sudo systemctl restart vault

# Check status
sudo systemctl status vault

# View logs
sudo journalctl -u vault -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add vault default

# Start service
rc-service vault start

# Stop service
rc-service vault stop

# Restart service
rc-service vault restart

# Check status
rc-service vault status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'vault_enable="YES"' >> /etc/rc.conf

# Start service
service vault start

# Stop service
service vault stop

# Restart service
service vault restart

# Check status
service vault status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start vault
brew services stop vault
brew services restart vault

# Check status
brew services list | grep vault
```

### Windows Service Manager

```powershell
# Start service
net start vault

# Stop service
net stop vault

# Using PowerShell
Start-Service vault
Stop-Service vault
Restart-Service vault

# Check status
Get-Service vault
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream vault_backend {
    server 127.0.0.1:8200;
}

server {
    listen 80;
    server_name vault.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name vault.example.com;

    ssl_certificate /etc/ssl/certs/vault.example.com.crt;
    ssl_certificate_key /etc/ssl/private/vault.example.com.key;

    location / {
        proxy_pass http://vault_backend;
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
    ServerName vault.example.com
    Redirect permanent / https://vault.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName vault.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/vault.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/vault.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8200/
    ProxyPassReverse / http://127.0.0.1:8200/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend vault_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/vault.pem
    redirect scheme https if !{ ssl_fc }
    default_backend vault_backend

backend vault_backend
    balance roundrobin
    server vault1 127.0.0.1:8200 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R vault:vault /etc/vault
sudo chmod 750 /etc/vault

# Configure firewall
sudo firewall-cmd --permanent --add-port=8200/tcp
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
sudo systemctl status vault

# View logs
sudo journalctl -u vault -f

# Monitor resource usage
top -p $(pgrep vault)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/vault"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/vault-backup-$DATE.tar.gz" /etc/vault /var/lib/vault

echo "Backup completed: $BACKUP_DIR/vault-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop vault

# Restore from backup
tar -xzf /backup/vault/vault-backup-*.tar.gz -C /

# Start service
sudo systemctl start vault
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u vault -n 100
sudo tail -f /var/log/vault/vault.log

# Check configuration
vault --version

# Check permissions
ls -la /etc/vault
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8200

# Test connectivity
telnet localhost 8200

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep vault)

# Check disk I/O
iotop -p $(pgrep vault)

# Check connections
ss -an | grep 8200
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  vault:
    image: vault:latest
    ports:
      - "8200:8200"
    volumes:
      - ./config:/etc/vault
      - ./data:/var/lib/vault
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update vault

# Debian/Ubuntu
sudo apt update && sudo apt upgrade vault

# Arch Linux
sudo pacman -Syu vault

# Alpine Linux
apk update && apk upgrade vault

# openSUSE
sudo zypper update vault

# FreeBSD
pkg update && pkg upgrade vault

# Always backup before updates
tar -czf /backup/vault-pre-update-$(date +%Y%m%d).tar.gz /etc/vault

# Restart after updates
sudo systemctl restart vault
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/vault

# Clean old logs
find /var/log/vault -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/vault
```

## Additional Resources

- Official Documentation: https://docs.vault.org/
- GitHub Repository: https://github.com/vault/vault
- Community Forum: https://forum.vault.org/
- Best Practices Guide: https://docs.vault.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

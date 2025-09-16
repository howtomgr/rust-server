# rust-server Installation Guide

rust-server is a free and open-source Rust game server. Rust survival game dedicated server

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
  - CPU: 4+ cores
  - RAM: 8GB minimum
  - Storage: 50GB for game
  - Network: Steam protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 28015 (default rust-server port)
  - RCON on 28016
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

# Install rust-server
sudo dnf install -y rust-server

# Enable and start service
sudo systemctl enable --now rust

# Configure firewall
sudo firewall-cmd --permanent --add-port=28015/tcp
sudo firewall-cmd --reload

# Verify installation
RustDedicated --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install rust-server
sudo apt install -y rust-server

# Enable and start service
sudo systemctl enable --now rust

# Configure firewall
sudo ufw allow 28015

# Verify installation
RustDedicated --version
```

### Arch Linux

```bash
# Install rust-server
sudo pacman -S rust-server

# Enable and start service
sudo systemctl enable --now rust

# Verify installation
RustDedicated --version
```

### Alpine Linux

```bash
# Install rust-server
apk add --no-cache rust-server

# Enable and start service
rc-update add rust default
rc-service rust start

# Verify installation
RustDedicated --version
```

### openSUSE/SLES

```bash
# Install rust-server
sudo zypper install -y rust-server

# Enable and start service
sudo systemctl enable --now rust

# Configure firewall
sudo firewall-cmd --permanent --add-port=28015/tcp
sudo firewall-cmd --reload

# Verify installation
RustDedicated --version
```

### macOS

```bash
# Using Homebrew
brew install rust-server

# Start service
brew services start rust-server

# Verify installation
RustDedicated --version
```

### FreeBSD

```bash
# Using pkg
pkg install rust-server

# Enable in rc.conf
echo 'rust_enable="YES"' >> /etc/rc.conf

# Start service
service rust start

# Verify installation
RustDedicated --version
```

### Windows

```bash
# Using Chocolatey
choco install rust-server

# Or using Scoop
scoop install rust-server

# Verify installation
RustDedicated --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/rust-server

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
RustDedicated --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable rust

# Start service
sudo systemctl start rust

# Stop service
sudo systemctl stop rust

# Restart service
sudo systemctl restart rust

# Check status
sudo systemctl status rust

# View logs
sudo journalctl -u rust -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add rust default

# Start service
rc-service rust start

# Stop service
rc-service rust stop

# Restart service
rc-service rust restart

# Check status
rc-service rust status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'rust_enable="YES"' >> /etc/rc.conf

# Start service
service rust start

# Stop service
service rust stop

# Restart service
service rust restart

# Check status
service rust status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start rust-server
brew services stop rust-server
brew services restart rust-server

# Check status
brew services list | grep rust-server
```

### Windows Service Manager

```powershell
# Start service
net start rust

# Stop service
net stop rust

# Using PowerShell
Start-Service rust
Stop-Service rust
Restart-Service rust

# Check status
Get-Service rust
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream rust-server_backend {
    server 127.0.0.1:28015;
}

server {
    listen 80;
    server_name rust-server.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name rust-server.example.com;

    ssl_certificate /etc/ssl/certs/rust-server.example.com.crt;
    ssl_certificate_key /etc/ssl/private/rust-server.example.com.key;

    location / {
        proxy_pass http://rust-server_backend;
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
    ServerName rust-server.example.com
    Redirect permanent / https://rust-server.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName rust-server.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/rust-server.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/rust-server.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:28015/
    ProxyPassReverse / http://127.0.0.1:28015/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend rust-server_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/rust-server.pem
    redirect scheme https if !{ ssl_fc }
    default_backend rust-server_backend

backend rust-server_backend
    balance roundrobin
    server rust-server1 127.0.0.1:28015 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R rust-server:rust-server /etc/rust-server
sudo chmod 750 /etc/rust-server

# Configure firewall
sudo firewall-cmd --permanent --add-port=28015/tcp
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
sudo systemctl status rust

# View logs
sudo journalctl -u rust -f

# Monitor resource usage
top -p $(pgrep rust-server)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/rust-server"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/rust-server-backup-$DATE.tar.gz" /etc/rust-server /var/lib/rust-server

echo "Backup completed: $BACKUP_DIR/rust-server-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop rust

# Restore from backup
tar -xzf /backup/rust-server/rust-server-backup-*.tar.gz -C /

# Start service
sudo systemctl start rust
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u rust -n 100
sudo tail -f /var/log/rust-server/rust-server.log

# Check configuration
RustDedicated --version

# Check permissions
ls -la /etc/rust-server
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 28015

# Test connectivity
telnet localhost 28015

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep rust-server)

# Check disk I/O
iotop -p $(pgrep rust-server)

# Check connections
ss -an | grep 28015
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  rust-server:
    image: rust-server:latest
    ports:
      - "28015:28015"
    volumes:
      - ./config:/etc/rust-server
      - ./data:/var/lib/rust-server
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update rust-server

# Debian/Ubuntu
sudo apt update && sudo apt upgrade rust-server

# Arch Linux
sudo pacman -Syu rust-server

# Alpine Linux
apk update && apk upgrade rust-server

# openSUSE
sudo zypper update rust-server

# FreeBSD
pkg update && pkg upgrade rust-server

# Always backup before updates
tar -czf /backup/rust-server-pre-update-$(date +%Y%m%d).tar.gz /etc/rust-server

# Restart after updates
sudo systemctl restart rust
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/rust-server

# Clean old logs
find /var/log/rust-server -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/rust-server
```

## Additional Resources

- Official Documentation: https://docs.rust-server.org/
- GitHub Repository: https://github.com/rust-server/rust-server
- Community Forum: https://forum.rust-server.org/
- Best Practices Guide: https://docs.rust-server.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.

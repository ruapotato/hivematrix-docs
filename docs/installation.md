# Installation Guide

This guide will walk you through installing HiveMatrix from scratch on a Linux system.

## System Requirements

### Minimum Requirements
- **OS**: Ubuntu 20.04 LTS or later (Debian-based Linux)
- **RAM**: 4GB minimum, 8GB recommended
- **CPU**: 2 cores minimum, 4 cores recommended
- **Disk**: 20GB free space minimum
- **Network**: Internet connection for package installation

### Software Dependencies (Auto-installed)
- Python 3.10 or later
- PostgreSQL 14 or later
- Neo4j 5.x (optional, for KnowledgeTree)
- Java 21 (for Keycloak)
- Git

## Quick Start

The recommended installation workflow:

```bash
# Create HiveMatrix directory
mkdir hivematrix
cd hivematrix

# Clone the orchestration system
git clone https://github.com/ruapotato/hivematrix-helm
cd hivematrix-helm

# Run the installation (installs core components)
./start.sh
```

The `start.sh` script will:

1. ✓ Check and install system dependencies
2. ✓ Set up Python virtual environment
3. ✓ Download and configure Keycloak
4. ✓ Clone Core and Nexus repositories
5. ✓ Set up PostgreSQL databases
6. ✓ Generate SSL certificates for HTTPS
7. ✓ Configure all services
8. ✓ Start the platform

After core installation completes, press **Ctrl+C** to stop `start.sh`.

### Installing Additional Modules

Now install the services you need:

```bash
# Install individual modules
sudo ./install_manager.py install codex       # Central data hub
sudo ./install_manager.py install ledger      # Billing system
sudo ./install_manager.py install knowledgetree  # Knowledge base (requires Neo4j)
sudo ./install_manager.py install brainhair   # AI assistant

# Or install all default apps
sudo ./install_manager.py install-defaults
```

Then restart to run with all installed modules:

```bash
./start.sh
```

## Step-by-Step Installation

### 1. Clone Helm Repository

```bash
cd ~/Work
mkdir -p hivematrix
cd hivematrix
git clone https://github.com/ruapotato/hivematrix-helm
cd hivematrix-helm
```

### 2. Run the Installer

```bash
./start.sh
```

The first run will take several minutes as it:
- Installs system packages
- Downloads Keycloak (~200MB)
- Clones repositories
- Sets up databases
- Generates configuration

### 3. Monitor Installation

The installer provides detailed output:

```
================================================================
  HiveMatrix Helm - Orchestration System
================================================================

Verifying HiveMatrix installation...

================================================================
  Step 1: System Dependencies
================================================================

Checking Python...
✓ Python 3.12.3

Checking Git...
✓ Git installed

... (continues with each step)
```

### 4. Post-Installation

After installation completes, you'll see:

```
================================================================
  HiveMatrix is Ready!
================================================================

  Login URL:         https://localhost:443
  Helm Dashboard:    http://localhost:5004

  Default Login:
    Username: admin
    Password: admin

================================================================
```

## Installation Options

### Installing Specific Services

By default, Core and Nexus are installed (required). Additional services are installed on-demand:

```bash
# The system will detect and offer to install additional services
# when you access them through the UI
```

Alternatively, clone services manually:

```bash
cd ~/Work/hivematrix

# Clone desired services
git clone https://github.com/ruapotato/hivematrix-codex
git clone https://github.com/ruapotato/hivematrix-ledger
git clone https://github.com/ruapotato/hivematrix-knowledgetree
git clone https://github.com/ruapotato/hivematrix-brainhair

# Restart to auto-detect new services
cd hivematrix-helm
./stop.sh
./start.sh
```

### Installing Neo4j (for KnowledgeTree)

If you want to use KnowledgeTree, install Neo4j:

```bash
# Run the Neo4j installation script
cd hivematrix-helm
sudo bash install_neo4j.sh
```

This will:
- Add Neo4j apt repository
- Install Neo4j Community Edition
- Configure Neo4j for HiveMatrix
- Set default password

## Verifying Installation

### Check Service Status

```bash
cd hivematrix-helm
source pyenv/bin/activate
python service_manager.py status
```

Expected output:

```
================================================================================
HiveMatrix Service Status
================================================================================

KEYCLOAK
  Status:  running
  Health:  healthy
  Port:    8080

CORE
  Status:  running
  Health:  healthy
  Port:    5000

NEXUS
  Status:  running
  Health:  healthy
  Port:    443

... (other services)
```

### Test Web Access

1. Open browser to https://localhost:443
2. Accept self-signed certificate warning (on first visit)
3. You should see the Keycloak login page
4. Log in with admin/admin

### Run Security Audit

```bash
cd hivematrix-helm
source pyenv/bin/activate
python security_audit.py --audit
```

This checks:
- Service port bindings
- Firewall status
- SSL certificates
- Database connections

## Troubleshooting Installation

### Port Already in Use

If you see "Address already in use" errors:

```bash
# Check what's using the port (e.g., port 443)
sudo lsof -i :443

# Stop the conflicting service or change HiveMatrix ports in services.json
```

### PostgreSQL Connection Refused

If PostgreSQL isn't running:

```bash
# Start PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Check status
sudo systemctl status postgresql
```

### Keycloak Won't Start

If Keycloak fails to start:

```bash
# Check Java version (must be 21)
java -version

# Check Keycloak logs
tail -f logs/keycloak.stderr.log

# Restart Keycloak
cd ../keycloak-26.4.0
./bin/kc.sh stop
./bin/kc.sh start-dev
```

### Permission Denied Errors

If you see permission errors:

```bash
# Fix ownership of HiveMatrix directory
cd ~/Work/hivematrix
sudo chown -R $USER:$USER .

# Fix git directory permissions
cd hivematrix-helm
sudo chown -R $USER:$USER .git
```

### Clean Installation

To start over from scratch:

```bash
# Stop all services
cd ~/Work/hivematrix/hivematrix-helm
./stop.sh

# Remove all data (WARNING: This deletes everything!)
sudo systemctl stop neo4j
sudo -u postgres dropdb --if-exists core_db
sudo -u postgres dropdb --if-exists helm_db
sudo -u postgres dropdb --if-exists codex_db
sudo -u postgres dropdb --if-exists ledger_db
sudo -u postgres dropdb --if-exists brainhair_db
sudo -u postgres dropdb --if-exists knowledgetree_db

# Remove Keycloak data
rm -rf ../keycloak-26.4.0/data/*

# Restart installation
./start.sh
```

## Autostart Configuration

HiveMatrix can be configured to start automatically on system boot using systemd.

### Enable Autostart

```bash
cd hivematrix-helm
sudo python3 autostart.py setup
```

This will:
- Create systemd service unit for HiveMatrix
- Configure it to start on boot
- Set up proper user permissions
- Enable the service

### Manage Autostart Service

```bash
# Check status
sudo systemctl status hivematrix

# Start/stop/restart
sudo systemctl start hivematrix
sudo systemctl stop hivematrix
sudo systemctl restart hivematrix

# View logs
sudo journalctl -u hivematrix -f

# Disable autostart
sudo python3 autostart.py remove
```

The autostart service will:
- Start all HiveMatrix services on boot
- Run as your user (not root)
- Automatically restart services if they crash
- Log to systemd journal

### Autostart Best Practices

1. **Test First**: Make sure HiveMatrix works with `./start.sh` before enabling autostart
2. **Monitor Logs**: Check `sudo journalctl -u hivematrix` for startup issues
3. **Database Dependencies**: Ensure PostgreSQL and Neo4j start before HiveMatrix
4. **Resource Limits**: The systemd unit includes appropriate memory/CPU limits

## Post-Installation Configuration

### Change Default Password

1. Log in with admin/admin
2. Click your name in top right
3. Go to "Account Settings"
4. Change password

### Configure Firewall

For production deployments:

```bash
cd hivematrix-helm
source pyenv/bin/activate

# Generate firewall rules
python security_audit.py --generate-firewall

# Review the script
cat secure_firewall.sh

# Apply firewall rules
sudo bash secure_firewall.sh
```

### Set Up SSL Certificate

For production with a real domain:

```bash
# Install certbot
sudo apt install certbot

# Get Let's Encrypt certificate
sudo certbot certonly --standalone -d your-domain.com

# Update Nexus to use the certificate
# Edit nexus config to point to /etc/letsencrypt/live/your-domain.com/
```

### Configure Backups

Set up automated backups:

```bash
# Create backup script
sudo crontab -e

# Add daily backup at 2 AM
0 2 * * * cd /home/your-user/Work/hivematrix/hivematrix-helm && /usr/bin/python3 backup.py >> /var/log/hivematrix-backup.log 2>&1
```

## Next Steps

- Read the [Architecture Guide](ARCHITECTURE.md) to understand the system
- Explore the [Services Overview](services/overview.md) to learn about each component
- Check the [Configuration Guide](configuration.md) for customization options
- Review the [Security Guide](security.md) for production deployment

## Getting Help

If you encounter issues:

1. Check the [troubleshooting section](#troubleshooting-installation) above
2. Review logs in `hivematrix-helm/logs/`
3. Run the security audit for configuration issues
4. Open an issue on GitHub with logs and error messages

---
icon: lucide/wrench
---

# Run as Linux service

You need to ensure the `garage` user has proper ownership and permissions for both data directories. Here's the corrected setup:

## Create Dedicated User and Directories

```bash
# Create dedicated user for Garage
sudo useradd --system --shell /bin/false --comment "Garage Object Store" garage

# Create the meta and data directories with proper structure
sudo mkdir -p /var/lib/garage/meta
sudo mkdir -p /var/lib/garage/data
sudo mkdir -p /var/log/garage

# Set ownership recursively
sudo chown -R garage:garage /var/lib/garage
sudo chown -R garage:garage /var/log/garage

# Set appropriate permissions
sudo chmod 750 /var/lib/garage
sudo chmod 750 /var/lib/garage/meta
sudo chmod 750 /var/lib/garage/data
sudo chmod 750 /var/log/garage
```

## Verify Directory Structure

```bash
# Check the directory structure and permissions
sudo ls -la /var/lib/garage/
```

You should see:
```
drwxr-x--- 4 garage garage 4096 Dec 1 12:00 .
drwxr-xr-x 1 root   root   4096 Dec 1 12:00 ..
drwxr-x--- 2 garage garage 4096 Dec 1 12:00 data
drwxr-x--- 2 garage garage 4096 Dec 1 12:00 meta
```

## Install Garage Binary

```bash
# Copy garage binary to standard location
sudo cp garage /usr/local/bin/
sudo chown root:root /usr/local/bin/garage
sudo chmod 755 /usr/local/bin/garage
```

## Set Permissions for Config File

```bash
# Ensure garage user can read the config file
sudo chown root:garage /etc/garage.toml
sudo chmod 640 /etc/garage.toml
```

## Create Systemd Service File

Create the service file with proper directory access:

```bash
sudo nano /etc/systemd/system/garage.service
```

**Content:**
```ini
[Unit]
Description=Garage Object Store
Documentation=https://garagehq.deuxfleurs.fr/
After=network.target
Wants=network.target

[Service]
Type=simple
User=garage
Group=garage
WorkingDirectory=/var/lib/garage
ExecStart=/usr/local/bin/garage -c /etc/garage.toml server
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5

# Security hardening
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/garage/meta /var/lib/garage/data /var/log/garage
ReadOnlyPaths=/etc/garage.toml
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes

# Resource limits (important for Garage)
LimitNOFILE=65536

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=garage

[Install]
WantedBy=multi-user.target
```

## Alternative: More Permissive Directory Access

If Garage needs to create additional directories, you can use:

```ini
[Service]
# ... other settings ...

# Give access to the entire garage directory
ReadWritePaths=/var/lib/garage /var/log/garage
ReadOnlyPaths=/etc/garage.toml

# Or be more specific:
# ReadWritePaths=/var/lib/garage/meta /var/lib/garage/data /var/log/garage
```

## Verify Your Config File

Make sure your `/etc/garage.toml` points to the correct directories:

```toml
metadata_dir = "/var/lib/garage/meta"
data_dir = "/var/lib/garage/data"

replication_mode = "none"

rpc_bind_addr = "[::]:3901"
rpc_public_addr = "[::]:3901"

[s3_api]
s3_region = "garage"
api_bind_addr = "[::]:3900"
root_domain = ".s3.garage"

[s3_web]
bind_addr = "[::]:3902"
root_domain = ".web.garage"
```

## Enable and Start the Service

```bash
# Reload systemd to recognize the new service
sudo systemctl daemon-reload

# Enable service to start on boot
sudo systemctl enable garage.service

# Start the service
sudo systemctl start garage.service

# Check status
sudo systemctl status garage.service

# Follow logs in real-time to see if there are any permission issues
sudo journalctl -u garage.service -f
```

## Verify Directory Access

After starting the service, check if Garage can access the directories:

```bash
# Check if Garage created files in the directories
sudo ls -la /var/lib/garage/meta/
sudo ls -la /var/lib/garage/data/

# Check ownership of any created files
sudo ls -la /var/lib/garage/meta/ | head -10

# Test garage functionality
sudo -u garage /usr/local/bin/garage status
```

## Troubleshooting Permission Issues

If you encounter permission issues:

```bash
# Check current permissions
sudo ls -la /var/lib/garage/

# Fix ownership if needed
sudo chown -R garage:garage /var/lib/garage

# Fix permissions if needed
sudo chmod 750 /var/lib/garage/meta
sudo chmod 750 /var/lib/garage/data

# Check logs for errors
sudo journalctl -u garage.service --since "5 minutes ago"
```

## Key Points for Directory Access:

1. **Ownership**: Both `/var/lib/garage/meta` and `/var/lib/garage/data` owned by `garage:garage`
2. **Permissions**: Directories have `750` permissions (user read/write/execute, group read/execute)
3. **Systemd Access**: `ReadWritePaths` explicitly includes both data directories
4. **Working Directory**: Set to `/var/lib/garage` so relative paths work correctly

This setup ensures Garage has full read/write access to both its metadata and data directories while maintaining proper security isolation.
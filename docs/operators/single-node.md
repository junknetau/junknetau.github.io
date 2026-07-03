---
icon: lucide/server
title: Garage as a Linux service
description: >
  Install Garage as a hardened systemd service with a dedicated user —
  the base configuration for every Junk Net node.
---

# Garage as a Linux service

This playbook installs [Garage](https://garagehq.deuxfleurs.fr/) on a
single machine as a hardened systemd service. It's both a standalone
single-node setup (fine for trying Garage out) and step 1 of building
a Junk Net node — the [mesh](nebula.md) and [cluster](cluster.md)
playbooks build on it.

!!! note "Single node vs. cluster config"

    The config below is for a **standalone node**
    (`replication_factor = 1`, SQLite, localhost RPC). When the node
    joins the Junk Net cluster, the config changes to the
    [cluster version](cluster.md) — same service, same directories,
    different `/etc/garage.toml`.

## Download Garage

Check [Garage downloads](https://garagehq.deuxfleurs.fr/download/) for
the latest release and your architecture:

``` bash
# v2.1.0 stable, for typical x86_64 laptops
curl -O https://garagehq.deuxfleurs.fr/_releases/v2.1.0/x86_64-unknown-linux-musl/garage
```

## Create the configuration file

``` bash
cat > garage.toml <<EOF
metadata_dir = "/var/lib/garage/meta"
data_dir = "/var/lib/garage/data"
db_engine = "sqlite"

replication_factor = 1

rpc_bind_addr = "[::]:3901"
rpc_public_addr = "127.0.0.1:3901"
rpc_secret = "$(openssl rand -hex 32)"

[s3_api]
s3_region = "garage"
api_bind_addr = "[::]:3900"
root_domain = ".s3.garage.localhost"

[s3_web]
bind_addr = "[::]:3902"
root_domain = ".web.garage.localhost"
index = "index.html"

[admin]
api_bind_addr = "[::]:3903"
admin_token = "$(openssl rand -base64 32)"
metrics_token = "$(openssl rand -base64 32)"
EOF

# Move it into place
sudo mv garage.toml /etc/garage.toml
```

The `$(openssl rand ...)` substitutions generate a unique RPC secret
and admin tokens as the file is written. Keep the admin token
somewhere safe — you'll need it for administration, and anyone who has
it controls the node.

## Create a dedicated user and directories

Garage runs as its own unprivileged system user, able to touch only
its own directories:

``` bash
# Dedicated system user, no shell, no home
sudo useradd --system --shell /bin/false --comment "Garage Object Store" garage

# Metadata and data directories
sudo mkdir -p /var/lib/garage/meta
sudo mkdir -p /var/lib/garage/data

# Owned by garage, readable by nobody else
sudo chown -R garage:garage /var/lib/garage
sudo chmod 750 /var/lib/garage /var/lib/garage/meta /var/lib/garage/data
```

Verify:

``` bash
sudo ls -la /var/lib/garage/
```

``` title="Expected"
drwxr-x--- 4 garage garage 4096 Jul  3 12:00 .
drwxr-xr-x 1 root   root   4096 Jul  3 12:00 ..
drwxr-x--- 2 garage garage 4096 Jul  3 12:00 data
drwxr-x--- 2 garage garage 4096 Jul  3 12:00 meta
```

## Install the binary and lock down the config

``` bash
# Binary: root-owned, world-executable
sudo mv garage /usr/local/bin/
sudo chown root:root /usr/local/bin/garage
sudo chmod 755 /usr/local/bin/garage

# Config: contains secrets — readable only by root and the garage group
sudo chown root:garage /etc/garage.toml
sudo chmod 640 /etc/garage.toml
```

## Create the systemd service

``` bash
sudo tee /etc/systemd/system/garage.service > /dev/null <<'EOF'
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

# Security hardening: the service sees a read-only system,
# with write access only to its own directories.
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/garage
ReadOnlyPaths=/etc/garage.toml
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes

# Garage keeps many block files open
LimitNOFILE=65536

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=garage

[Install]
WantedBy=multi-user.target
EOF
```

The hardening lines mean a compromised Garage process can't write
anywhere except `/var/lib/garage`, can't see home directories, and
can't gain privileges — appropriate paranoia for a service holding
other people's data on donated hardware.

## Enable and start

``` bash
sudo systemctl daemon-reload
sudo systemctl enable --now garage.service

# Did it come up?
sudo systemctl status garage.service

# Watch the logs live
sudo journalctl -u garage.service -f
```

## Verify

``` bash
# Garage created its metadata and data structures?
sudo ls -la /var/lib/garage/meta/
sudo ls -la /var/lib/garage/data/

# The node answers?
sudo garage -c /etc/garage.toml status
```

`garage status` should show one healthy node (this machine) with a
node ID — that ID is what the [cluster playbook](cluster.md) will use.

### Optional: smoke-test with a real upload

On a standalone node you can assign a trivial layout and push a file
through the S3 API end to end:

``` bash
NODE=$(sudo garage -c /etc/garage.toml status | awk 'NR>2 {print $1; exit}')
sudo garage -c /etc/garage.toml layout assign "$NODE" -z test -c 10GB
sudo garage -c /etc/garage.toml layout apply --version 1

sudo garage -c /etc/garage.toml bucket create test-bucket
sudo garage -c /etc/garage.toml key create test-key      # note the key ID + secret
sudo garage -c /etc/garage.toml bucket allow test-bucket --read --write --key test-key
```

Then upload with any S3 client pointed at `http://localhost:3900`
(region `garage`) — see [Using your storage](../use/index.md) for
client configs.

## Troubleshooting

``` bash
# Permission errors on startup? Check ownership and modes:
sudo ls -la /var/lib/garage/
sudo chown -R garage:garage /var/lib/garage
sudo chmod 750 /var/lib/garage/meta /var/lib/garage/data

# Config not readable? (needs root:garage 640)
sudo ls -la /etc/garage.toml

# Read the last few minutes of logs
sudo journalctl -u garage.service --since "5 minutes ago"
```

The usual culprits, in order: directory ownership, config file
permissions, another process on ports 3900–3903, or a typo in
`/etc/garage.toml` (Garage says exactly which line it hates).

## Key points

1. **Dedicated user** — Garage never runs as root.
2. **`750` directories, `640` config** — the RPC secret and admin
   token are not world-readable.
3. **`ProtectSystem=strict` + `ReadWritePaths`** — the service can
   write only to `/var/lib/garage`, even if compromised.
4. **`Restart=on-failure`** — on a community node nobody's watching,
   the service must pick itself back up.

Next: [give the node its mesh address](nebula.md), then
[join it to the cluster](cluster.md).

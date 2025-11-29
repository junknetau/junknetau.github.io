---
icon: lucide/boxes
---


# Multi node notes

## Sample /etc/garage.toml file

```bash
# Generate a valid garage.toml but replace rpc_public_addr and rpc_secret
cat > garage.toml <<EOF
metadata_dir = "/var/lib/garage/meta"
data_dir = "/var/lib/garage/data"
db_engine = "lmdb"
metadata_auto_snapshot_interval = "6h"

replication_factor = 3

compression_level = 2

rpc_bind_addr = "[::]:3901"
rpc_public_addr = "<this node's public IP>:3901"
rpc_secret = "<RPC secret>"

[s3_api]
s3_region = "garage"
api_bind_addr = "[::]:3900"
root_domain = ".s3.garage"

[s3_web]
bind_addr = "[::]:3902"
root_domain = ".web.garage"
index = "index.html"
EOF

# Move to the default config directory
sudo mv garage.toml /etc/garage.toml

# Set an alias for garage commands
alias garage="sudo /usr/local/bin/garage -c /etc/garage.toml"

# Also just for fun
alias junknet="sudo /usr/local/bin/garage -c /etc/garage.toml"
```

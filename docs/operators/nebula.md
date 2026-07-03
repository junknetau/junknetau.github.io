---
icon: lucide/network
title: Set up the mesh
description: >
  Join a node to the Junk Net Nebula mesh — certificates, config,
  lighthouse, and the systemd service.
---

# Set up the mesh

This playbook connects a node to the Junk Net
[Nebula](https://github.com/slackhq/nebula) mesh, giving it a
permanent private IP (e.g. `10.42.1.11`) that works from any home,
behind any router. The [mesh architecture page](../how-it-works/mesh.md)
explains *why* it works; this page is the *how*.

!!! info "Who signs what"

    Nodes generate their own keys, but certificates are signed by the
    project CA, which lives offline with the maintainer. In practice:
    you run the keygen step, send the **public** key over, and get a
    signed certificate back. The CA key never touches a node. (For a
    from-scratch community cluster, the CA creation step is included
    below.)

## Install Nebula

Grab the latest release for your architecture from
[Nebula's releases page](https://github.com/slackhq/nebula/releases):

``` bash
curl -LO https://github.com/slackhq/nebula/releases/latest/download/nebula-linux-amd64.tar.gz
tar -xzf nebula-linux-amd64.tar.gz

sudo mv nebula /usr/local/bin/
sudo mv nebula-cert /usr/local/bin/
sudo chown root:root /usr/local/bin/nebula /usr/local/bin/nebula-cert
sudo chmod 755 /usr/local/bin/nebula /usr/local/bin/nebula-cert

sudo mkdir -p /etc/nebula
```

## One-time: create the CA (cluster founders only)

Done **once per community**, on a machine that is not a node, and the
resulting `ca.key` is kept offline:

``` bash
nebula-cert ca -name "Junk Net Brisbane" -duration 87600h   # 10 years
# Produces ca.crt (public — distributed to every node)
#      and ca.key (PRIVATE — offline, guarded, never on a node)
```

## One-time: stand up a lighthouse

The lighthouse is a tiny Nebula node on a stable public IP — the
cheapest VPS you can find is plenty (it coordinates; it doesn't carry
storage traffic). Sign it a certificate in the lighthouse range and
give it a config with `am_lighthouse: true`:

``` bash
# On the CA machine:
nebula-cert sign -name "lighthouse-1" -ip "10.42.0.1/16" -groups "lighthouse"
```

``` yaml title="/etc/nebula/config.yml (lighthouse)"
pki:
  ca: /etc/nebula/ca.crt
  cert: /etc/nebula/lighthouse-1.crt
  key: /etc/nebula/lighthouse-1.key

static_host_map: {}

lighthouse:
  am_lighthouse: true

listen:
  host: 0.0.0.0
  port: 4242          # open UDP 4242 in the VPS firewall

punchy:
  punch: true

relay:
  am_relay: true      # fallback path for hostile NATs

firewall:
  outbound:
    - port: any
      proto: any
      host: any
  inbound:
    - port: any
      proto: icmp
      host: any
```

## Per-node: keys and certificate

On the new node, generate a keypair; the CA signs the public half with
the node's permanent mesh IP and its group:

``` bash
# On the node — the private key never leaves this machine
nebula-cert keygen -out-key node.key -out-pub node.pub

# On the CA machine, against the received node.pub
# (IP allocated from the node range; groups drive the firewall)
nebula-cert sign -ca-crt ca.crt -ca-key ca.key \
  -name "node-annerley-thinkpad" -ip "10.42.1.11/16" \
  -groups "node" -in-pub node.pub
```

Install the three PKI files on the node:

``` bash
sudo mv node.key node-annerley-thinkpad.crt ca.crt /etc/nebula/
sudo chown root:root /etc/nebula/*
sudo chmod 600 /etc/nebula/node.key
sudo chmod 644 /etc/nebula/*.crt
```

## Per-node: configuration

``` yaml title="/etc/nebula/config.yml (storage node)"
pki:
  ca: /etc/nebula/ca.crt
  cert: /etc/nebula/node-annerley-thinkpad.crt
  key: /etc/nebula/node.key

# Where to find the lighthouse in the real world
static_host_map:
  "10.42.0.1": ["<lighthouse-public-ip>:4242"]

lighthouse:
  am_lighthouse: false
  interval: 60
  hosts:
    - "10.42.0.1"

listen:
  host: 0.0.0.0
  port: 0             # random port — kinder to home NATs

punchy:
  punch: true          # keep NAT mappings alive
  respond: true        # punch back for hard NATs

relay:
  relays:
    - 10.42.0.1        # fall back to the lighthouse if punching fails
  use_relays: true

tun:
  dev: nebula1

firewall:
  conntrack:
    tcp_timeout: 12m
    udp_timeout: 3m
    default_timeout: 10m

  outbound:
    - port: any
      proto: any
      host: any

  inbound:
    # Anyone on the mesh may ping us (debugging sanity)
    - port: any
      proto: icmp
      host: any
    # Fellow storage nodes may reach Garage's inter-node RPC
    - port: 3901
      proto: tcp
      groups:
        - node
    # Gateways may reach the S3 API
    - port: 3900
      proto: tcp
      groups:
        - gateway
    # Admin machines may reach everything Garage exposes
    - port: 3900-3903
      proto: tcp
      groups:
        - admin
```

That firewall is the whole security posture in one block: Garage's
ports are reachable only by certificate-holding members of the right
groups, and nothing else is reachable at all.

## Run it as a service

``` bash
sudo tee /etc/systemd/system/nebula.service > /dev/null <<'EOF'
[Unit]
Description=Nebula mesh (Junk Net)
Documentation=https://github.com/slackhq/nebula
Wants=network-online.target
After=network-online.target

[Service]
Type=notify
ExecStart=/usr/local/bin/nebula -config /etc/nebula/config.yml
Restart=on-failure
RestartSec=5
SyslogIdentifier=nebula

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now nebula.service
```

Order matters at boot: Garage binds its RPC address on the mesh, so
make Garage start after Nebula by adding to the garage unit:

``` bash
sudo systemctl edit garage.service
```

``` ini
[Unit]
After=nebula.service
Wants=nebula.service
```

## Verify

``` bash
# The mesh interface exists and has our IP?
ip addr show nebula1

# Can we reach the lighthouse across the mesh?
ping -c 3 10.42.0.1

# Can we reach another node? (ask an operator for a live node IP)
ping -c 3 10.42.1.12

# Logs, if anything is unhappy
sudo journalctl -u nebula.service -f
```

First ping to a new peer may take a beat (that's the hole-punch
handshake); after that, traffic flows directly between homes.

## Point Garage at the mesh

The payoff — in `/etc/garage.toml`, the node advertises its Nebula IP:

``` toml
rpc_bind_addr = "[::]:3901"
rpc_public_addr = "10.42.1.11:3901"    # this node's mesh IP
```

``` bash
sudo systemctl restart garage.service
```

From here the node is reachable by its peers and ready to
[join the cluster](cluster.md).

## Troubleshooting

- **`nebula1` missing** — the service didn't start; check
  `journalctl -u nebula.service` for cert path or YAML errors.
- **Can ping lighthouse but not peers** — usually a hard NAT; confirm
  `punchy.respond: true` on both ends and that relays are configured.
  Traffic falling back to relay shows in the logs.
- **Cert errors** — name, IP, and CA must all match what was signed;
  `nebula-cert print -path /etc/nebula/<node>.crt` shows what a cert
  actually says.
- **Garage can't bind after reboot** — Garage raced Nebula; confirm
  the `After=nebula.service` drop-in from above is in place.

---
icon: lucide/network
title: The mesh layer — Nebula
description: >
  How Junk Net nodes in different homes find each other and talk
  securely, using Nebula's certificate-based overlay network.
---

# The mesh layer: Nebula

Junk Net nodes live in different houses, behind different routers, on
different ISPs. None of them has a public IP address, and no volunteer
should ever be asked to port-forward their home router for a storage
cluster. The mesh layer solves this.

[Nebula](https://github.com/slackhq/nebula) is an open source overlay
network originally built at Slack to connect their global fleet. It
gives every Junk Net node a stable private IP on a virtual network
(e.g. `10.42.0.0/16`) that works the same whether the machine is in
Annerley or airport wifi.

## Why Nebula (and not just WireGuard)

WireGuard is an excellent *tunnel*, but it's point-to-point: a
full mesh of N nodes means N² manually-managed peer configs, and it has
no built-in answer for NAT traversal or authorization. Nebula is built
*on* the same Noise protocol cryptography, and adds the three things a
community mesh actually needs:

1.  **A certificate authority instead of peer lists.** The project runs
    a CA. Each node gets one signed certificate stating its identity,
    its mesh IP, and its groups (e.g. `junknet-node`). Nodes then
    authenticate *each other* directly — adding node #41 doesn't
    require touching the other 40 configs.

2.  **NAT traversal via lighthouses.** A *lighthouse* is a small Nebula
    node on a stable public address (a $5 VPS does fine). It stores no
    data and relays traffic only as a last resort — its job is
    introductions: it tells nodes each other's current public
    address/port so they can UDP hole-punch and talk **directly**,
    peer-to-peer. Storage traffic between Home A and Home B does not
    flow through the lighthouse.

3.  **A built-in, identity-based firewall.** Nebula rules are written
    against certificate groups, not IPs. Junk Net's policy is one line
    of intent: *nodes may reach Garage's ports on other nodes; nothing
    else may reach anything.*

The result: everything is encrypted end-to-end, every connection is
mutually authenticated against the CA, and a machine without a signed
certificate cannot even ping a node — let alone reach Garage.

## Junk Net's mesh design

``` mermaid
graph TB
    L[Lighthouse<br>small VPS, public IP<br>10.42.0.1]

    subgraph Home A
        N1[Node<br>10.42.1.11]
    end
    subgraph Home B
        N2[Node<br>10.42.1.12]
    end
    subgraph Home C
        N3[Node<br>10.42.1.13]
    end

    N1 -. "where is .12?" .-> L
    N2 -. "where is .13?" .-> L
    N3 -. announces itself .-> L

    N1 <==>|direct, encrypted UDP| N2
    N2 <==>|direct, encrypted UDP| N3
    N1 <==>|direct, encrypted UDP| N3
```

**Addressing.** The mesh is `10.42.0.0/16`. Lighthouses live in
`10.42.0.0/24`; storage nodes are allocated from `10.42.1.0/24`
onwards. A node's Nebula IP is permanent — it's baked into its
certificate — which makes it the natural value for Garage's
`rpc_public_addr`.

**Groups.** Certificates carry groups that the firewall keys off:

| Group | Who | May receive |
|-------|-----|-------------|
| `lighthouse` | Lighthouses | Nebula coordination traffic only |
| `node` | Storage nodes | Garage RPC (3901) from `node`; S3 (3900) and admin (3903) from `gateway` |
| `gateway` | Public S3 endpoint(s) | HTTPS from the internet (outside Nebula's scope) |
| `admin` | Maintainer machines | Everything, for operations |

**Certificate lifecycle.** When a laptop is provisioned it generates a
keypair locally; the CA signs a certificate binding its name, mesh IP,
and groups. Certificates expire and are renewed; a compromised or
retired node's certificate is added to the CA's blocklist and the mesh
stops talking to it. The CA private key never touches a storage node.

## How Garage rides the mesh

Garage doesn't know Nebula exists — it just sees a network interface:

``` toml title="/etc/garage.toml"
rpc_bind_addr = "[::]:3901"
rpc_public_addr = "10.42.1.11:3901"   # this node's Nebula IP
```

Because `rpc_public_addr` is a mesh address, nodes can *only* reach
each other through Nebula. Garage's own RPC secret then acts as a
second, independent authentication layer — belt and braces.

## Failure modes, honestly

- **Lighthouse down:** existing node-to-node tunnels keep working
  (they're direct); new connections and roaming nodes can't be
  introduced until it's back. Mitigation: lighthouses are trivial and
  cheap, so run two.
- **Hostile NAT (symmetric/CGNAT):** some connections can't be
  hole-punched and fall back to relaying through a lighthouse or
  another reachable node — slower, but functional. Relay hosts are
  configured explicitly.
- **CA key compromise:** the catastrophic one. The CA key lives
  offline, and rotating to shared custody of it is a
  [roadmap](../about/roadmap.md) milestone.

Setup instructions live in the operator docs:
[Set up the mesh](../operators/nebula.md).

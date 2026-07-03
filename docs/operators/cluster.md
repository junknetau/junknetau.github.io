---
icon: lucide/boxes
title: Cluster operations
description: >
  Join nodes into the Junk Net cluster — connect peers, assign zones
  and weights, apply the layout, and handle node loss gracefully.
---

# Cluster operations

With [Garage installed](single-node.md) and the node
[on the mesh](nebula.md), this playbook joins it to the cluster and
covers day-2 operations: growing the layout, draining nodes, and
handling the inevitable dead laptop.

## The cluster configuration

Cluster nodes replace the standalone `/etc/garage.toml` with this. Two
values are cluster-wide and must be identical everywhere:
`rpc_secret` (ask an operator) and `replication_factor = 3`.

``` bash
cat > garage.toml <<EOF
metadata_dir = "/var/lib/garage/meta"
data_dir = "/var/lib/garage/data"
db_engine = "lmdb"
metadata_auto_snapshot_interval = "6h"

replication_factor = 3

compression_level = 2

rpc_bind_addr = "[::]:3901"
rpc_public_addr = "<this node's Nebula IP>:3901"
rpc_secret = "<the cluster RPC secret>"

[s3_api]
s3_region = "garage"
api_bind_addr = "[::]:3900"
root_domain = ".s3.garage"

[s3_web]
bind_addr = "[::]:3902"
root_domain = ".web.garage"
index = "index.html"

[admin]
api_bind_addr = "[::]:3903"
admin_token = "<the cluster admin token>"
metrics_token = "<the cluster metrics token>"
EOF

sudo mv garage.toml /etc/garage.toml
sudo chown root:garage /etc/garage.toml
sudo chmod 640 /etc/garage.toml
sudo systemctl restart garage.service
```

Differences from the standalone config, and why:

- **`db_engine = "lmdb"`** — better suited than SQLite to the
  concurrent metadata traffic of a real cluster.
- **`metadata_auto_snapshot_interval = "6h"`** — periodic metadata
  snapshots, cheap insurance on elderly disks.
- **`replication_factor = 3`** — the durability promise.
- **`compression_level = 2`** — zstd on every block; residential
  upload bandwidth is the scarce resource, so spend CPU to save it.
- **`rpc_public_addr` = the Nebula IP** — peers reach this node only
  across the mesh.

Save some typing for everything below:

``` bash
# The official alias
alias garage="sudo /usr/local/bin/garage -c /etc/garage.toml"

# Also, just for fun
alias junknet="sudo /usr/local/bin/garage -c /etc/garage.toml"
```

## Connect the node to its peers

Every Garage node has an identity derived from its keypair. Get the
new node's full identifier:

``` bash
garage node id
# e.g. 563e1ac825ee3323aa441e72c26d1030d6d4414aeb3dd25287c531e7fc2bc95d@10.42.1.11:3901
```

Then introduce it to any existing cluster member (either direction
works — gossip spreads the word):

``` bash
# On an existing node, using the new node's full id@address:
garage node connect 563e1ac8...@10.42.1.11:3901
```

`garage status` on any node should now list the newcomer — connected,
but with no role yet:

``` text
==== HEALTHY NODES ====
ID                Hostname   Address           Tags  Zone  Capacity
563e1ac8…         thinkpad   10.42.1.11:3901         NO ROLE ASSIGNED
84f0aa9d…         macbook    10.42.1.12:3901   []    home-westend   250 GB
…
```

## Assign it a place in the layout

The *layout* is the declarative map of who stores what. Each node gets
a **zone** (in Junk Net: the household it lives in) and a **capacity**
(how much disk it offers):

``` bash
garage layout assign 563e1ac8 -z home-annerley -c 500GB
```

!!! tip "Zones are the durability story"

    Garage places each object's three replicas in **three distinct
    zones**. Name zones by household (`home-annerley`,
    `home-westend`, …), and keep at least three zones in the layout —
    with fewer, Garage can't honour `replication_factor = 3` across
    zones. Two laptops in the same house share one zone: honest
    accounting, since they share a power point and a flood plain.

Review what would change, then commit it:

``` bash
garage layout show          # staged changes + a new version number
garage layout apply --version <N>
```

Garage immediately starts rebalancing data toward the new node. Watch
it work:

``` bash
garage status               # data partitions assigned per node
garage worker list          # resync/rebalance queues draining
```

Rebalancing happens over home internet connections — a new node
filling up can take days. That's fine. It's junk; it's not in a hurry.

## Growing pains, planned for

=== "Adding a node"

    The happy path — exactly what you just did: `node connect`,
    `layout assign`, `layout apply`. Do it one node (or one batch) at
    a time and let rebalancing settle between rounds.

=== "Removing a node (planned)"

    A host is moving, or a laptop is being retired with dignity:

    ``` bash
    garage layout remove 563e1ac8
    garage layout apply --version <N>
    ```

    Garage drains the node — re-replicating its blocks elsewhere —
    while all data stays fully available. Once `garage status` shows
    the node holding nothing, power it off. **Drain before unplug**
    is the one etiquette rule of node hosting.

=== "Losing a node (unplanned)"

    A disk dies, a laptop grows legs. Nothing is lost — two replicas
    of everything it held are elsewhere. Restore full redundancy:

    ``` bash
    garage status                    # the node shows as failed
    garage layout remove <node-id>   # take it out of the layout
    garage layout apply --version <N>
    ```

    Garage re-replicates from the surviving copies back to three.
    If several nodes look "failed" at once, **stop** — that's usually
    a mesh problem, not a hardware apocalypse. Fix
    [Nebula](nebula.md#troubleshooting) first; never remove nodes that
    are merely unreachable.

## Cluster health checklist

The five commands that answer "is everything okay?":

``` bash
garage status            # who's here, who's missing, layout roles
garage stats             # storage used, per-node and cluster-wide
garage worker list       # background queues (resync, scrub) draining?
garage block list-errors # blocks the cluster couldn't fetch (want: none)
garage repair --yes blocks   # belt-and-braces integrity pass (slow, safe)
```

Plus the metadata snapshots you get for free from
`metadata_auto_snapshot_interval` — they land under
`/var/lib/garage/meta/snapshots/` and are the thing you'll want if a
node's metadata db is ever corrupted.

## Buckets and keys (cluster admin)

Contributor onboarding, from any node:

``` bash
garage bucket create <name>-bucket
garage key create <name>-key            # records the key ID + secret
garage bucket allow <name>-bucket --read --write --key <name>-key

# Keep allocations honest
garage bucket set-quotas <name>-bucket --max-size 300GB
```

Hand over the key ID, secret, and endpoint —
[Using your storage](../use/index.md) takes it from there.

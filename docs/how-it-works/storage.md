---
icon: lucide/hard-drive
title: The storage layer — Garage
description: >
  Why Junk Net uses Garage, and how it keeps data safe on old,
  unreliable hardware.
---

# The storage layer: Garage

[Garage](https://garagehq.deuxfleurs.fr/) is an open source,
S3-compatible object store built by [Deuxfleurs](https://deuxfleurs.fr/),
a French non-profit that runs its own services on second-hand machines
in members' homes. That origin story matters: Garage wasn't adapted to
Junk Net's situation — it was *designed* for it.

## Why Garage

Most distributed storage systems (MinIO, Ceph, SeaweedFS) assume a data
centre: matched hardware, sub-millisecond links, disks that rarely
vanish without warning. Junk Net has none of that. Garage's design
assumptions line up with ours almost exactly:

| Junk Net reality | Garage design answer |
|------------------|---------------------|
| Mismatched disks (120 GB netbook next to 1 TB ThinkPad) | Nodes are weighted by capacity; the layout algorithm gives each node work proportional to its size |
| High, variable latency between homes | No consensus protocol on the data path — CRDTs and quorum reads/writes tolerate slow links |
| Nodes disappear (unplugged, moved, dead) | Designed for churn: reads/writes succeed with 2 of 3 replicas; healing restores the third |
| Homes, not racks | First-class **zones** ensure replicas land in different failure domains |
| Volunteer operators | Single static binary, one TOML file, a friendly CLI |

## How Garage stores your data

**Objects become blocks.** Each uploaded object is split into blocks
(1&nbsp;MiB by default), addressed by hash and compressed with zstd.
Deduplication falls out for free: identical blocks are stored once.

**Blocks are placed by the layout.** The *cluster layout* is a
declarative table of every node: its identifier, its zone (in Junk Net,
the household it lives in), and its weight (its usable capacity).
From this, Garage computes an optimal assignment of data to nodes such
that every block has `replication_factor = 3` copies in **three
distinct zones**.

**Metadata is a CRDT store.** Bucket contents, object versions, and
access keys live in Garage's internal metadata engine (LMDB in our
multi-node setup), replicated with quorum semantics: a write is
acknowledged when 2 of 3 metadata replicas have it. There's no
leader and no election — any node can serve any request.

**Healing is automatic and continuous.** Garage constantly scrubs and
compares replicas (using Merkle trees, so comparison is cheap). If a
node is gone or a block is corrupt, it re-replicates from the surviving
copies. A `resync` queue drains whenever the cluster has spare
bandwidth.

## What this means on junk hardware

**A dying disk is an event, not an emergency.** With three
zone-separated copies, the failure procedure is: notice (SMART alerts,
`garage status`), remove the node from the layout, let the cluster
re-replicate, recycle the corpse. Data stays available throughout.

**Capacity math.** The cluster's usable capacity is roughly one third
of the raw disks (three copies of everything) — call it 30% after
overhead. Thirty donated laptops averaging 300&nbsp;GB each is ~9&nbsp;TB
raw, or about 3&nbsp;TB of genuinely durable community storage. Free.

**Bandwidth is the scarce resource.** Every write is sent to three
homes; every heal after a node loss moves that node's share of data
across residential upload links. This is why Junk Net's honest use
cases are backup and archival, and why the node image will throttle
resync traffic to stay polite on hosts' connections.

## Configuration highlights

The full setup lives in the [operator docs](../operators/cluster.md);
these are the choices that define Junk Net's cluster:

``` toml title="/etc/garage.toml (the parts that matter)"
db_engine = "lmdb"                        # metadata engine for multi-node
metadata_auto_snapshot_interval = "6h"    # periodic metadata snapshots
replication_factor = 3                    # three copies, non-negotiable
compression_level = 2                     # zstd on every block
rpc_bind_addr = "[::]:3901"               # inter-node RPC...
rpc_public_addr = "<nebula-ip>:3901"      # ...reachable only via the mesh
```

And the layout is where the community topology lives — one zone per
household:

``` bash
garage layout assign <node-id> -z home-annerley  -c 500GB
garage layout assign <node-id> -z home-westend   -c 250GB
garage layout assign <node-id> -z home-chermside -c 1TB
garage layout apply --version 1
```

## What Garage doesn't do (and what we do about it)

- **Encryption at rest.** Garage compresses and chunks but does not
  encrypt stored blocks. Junk Net's answer is client-side encryption —
  see [Trust & safety](trust.md).
- **Erasure coding.** Garage uses full replication (3× storage cost)
  rather than erasure codes. On heterogeneous, churny hardware this is
  the right trade: healing is simpler, reads don't fan out, and any
  single surviving replica is a complete copy.
- **Multi-cluster federation.** One Junk Net community = one Garage
  cluster. Linking communities is a someday-problem — see the
  [roadmap](../about/roadmap.md).

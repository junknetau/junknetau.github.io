---
icon: lucide/cog
title: Architecture
description: >
  How Junk Net works — donated laptops, a Nebula mesh, and a Garage
  object store, layer by layer.
---

# How it works

Junk Net is three layers, each doing one job:

| Layer | Made of | Job |
|-------|---------|-----|
| **Hardware** | Donated laptops | Provide disks, CPU, and a built-in UPS (the battery) |
| **Network** | [Nebula](mesh.md) mesh | Connect nodes across homes, NATs, and ISPs — encrypted, peer-to-peer |
| **Storage** | [Garage](storage.md) cluster | Pool the disks into one replicated, S3-compatible object store |

A user never sees any of this. They see an S3 endpoint and a pair of
access keys, and use it like any other object store.

## The big picture

``` mermaid
graph TB
    subgraph You
        C[S3 client<br>rclone / AWS CLI / SDK]
    end

    C -->|HTTPS, S3 API| G[Gateway node<br>public endpoint]

    subgraph Nebula mesh — encrypted overlay network
        G <--> N1
        G <--> N2
        G <--> N3
        N1[Node: old ThinkPad<br>Home A] <--> N2[Node: old MacBook<br>Home B]
        N2 <--> N3[Node: old Dell<br>Home C]
        N1 <--> N3
    end

    L[Lighthouse<br>small public host] -.NAT traversal
    coordination.-> N1
    L -.-> N2
    L -.-> N3
    L -.-> G
```

Every laptop runs the same small stack: Linux, the `nebula` daemon, and
the `garage` daemon — both as hardened systemd services. Nodes find and
talk to each other **only** over the mesh; Garage's ports are never
exposed to a home network or the public internet.

## Life of an object

Here's what happens when you upload `holiday-photos.zip`:

1.  **You `PUT` the object** to the S3 endpoint with your access keys.

2.  **Garage chunks it.** The object is split into blocks (1&nbsp;MiB by
    default), each identified by its hash. Blocks are compressed with
    zstd.

3.  **Garage places three copies of every block**, using the cluster
    layout to pick three nodes in **three different zones**. In Junk
    Net, a zone is a household — so your photos survive not just a dead
    disk, but a whole house losing power, internet, or existence.

4.  **Metadata is replicated too.** Which blocks make up which object,
    bucket listings, access keys — all stored on three nodes, kept
    consistent with quorum reads and writes (2 of 3 must agree).

5.  **You `GET` it back** from any gateway; Garage fetches blocks from
    whichever replicas are alive and reassembles the object.

The upshot: **any single node can vanish at any moment** — unplugged,
stolen, dropped in a bathtub — and nothing is lost. When a node stays
gone, Garage re-replicates its blocks onto other machines to get back
to three copies.

## Design decisions that matter

**A zone is a household.** Garage lets you tag each node with a zone
and guarantees replicas land in distinct zones. Junk Net uses this to
turn "three copies" into "three copies *in three different buildings*"
— which is the difference between surviving a disk failure and
surviving a burglary.

**The mesh is the only network.** Garage nodes authenticate to each
other with a shared RPC secret, but in Junk Net they also can't even
*reach* each other except through Nebula, which requires a certificate
signed by the project CA. Two independent locks.

**Old laptops are a feature, not a compromise.** Laptops idle at
10–25&nbsp;W, have built-in batteries that ride out power blips, and
are the single most commonly retired form factor. Garage weights each
node by its disk size, so a 2012 MacBook with 500&nbsp;GB and a netbook
with 120&nbsp;GB can serve in the same cluster, each pulling their own
weight.

**Boring tools, deliberately.** Debian, systemd, Garage, Nebula. Every
piece is open source, documented, and debuggable by a curious volunteer
at a kitchen table. No Kubernetes, no blockchain, nothing that needs a
team to babysit.

## Dig deeper

<div class="grid cards" markdown>

-   :lucide-hard-drive:{ .lg .middle } __The storage layer__

    ---

    How Garage replicates, rebalances, and survives flaky junk
    hardware.

    [:lucide-arrow-right: Garage in depth](storage.md)

-   :lucide-network:{ .lg .middle } __The mesh layer__

    ---

    How Nebula gets laptops in different homes talking through NAT,
    with certificates instead of port-forwarding.

    [:lucide-arrow-right: Nebula in depth](mesh.md)

-   :lucide-shield:{ .lg .middle } __Trust & safety__

    ---

    What node hosts can and can't see, how to encrypt client-side,
    and who holds the keys.

    [:lucide-arrow-right: The trust model](trust.md)

-   :lucide-wrench:{ .lg .middle } __Run a node__

    ---

    The operator playbooks: single node, cluster, and mesh setup.

    [:lucide-arrow-right: Operator docs](../operators/index.md)

</div>

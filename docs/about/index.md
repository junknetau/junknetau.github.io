---
icon: lucide/lightbulb
title: What is Junk Net?
description: >
  Junk Net is a community project that turns donated old laptops into a
  free, distributed, S3-compatible object store.
---

# What is Junk Net?

Junk Net is a community project with a simple trade at its heart:

> **Give us the laptop you're not using. Get free storage in return.**

Almost every household has one — a laptop that got replaced, got slow,
or got a cracked screen, and has been sitting in a drawer ever since.
It still works. It still has a perfectly good disk in it. It's just
junk now.

Junk Net collects those machines, securely wipes them, and installs an
OS image that turns each one into a **storage node**. The nodes connect
to each other over an encrypted mesh network and pool their disks into
one big, replicated, S3-compatible object store — a community cloud
built entirely from junk.

## What you get

If you contribute a laptop (or host a node at your place), you get:

- **Free S3-compatible object storage** — an endpoint and access keys
  that work with standard tools: `rclone`, the AWS CLI, Cyberduck, or
  any S3 client library.
- **Replicated durability** — every object is stored as three copies on
  different machines in different homes, so one dead laptop doesn't
  lose anything.
- **A stake in community infrastructure** — the cluster belongs to the
  people who built it, not to a company that can change the price or
  shut it down.

## The principles

Junk Net runs on a few non-negotiables:

1.  **Free means free.** No tiers, no trials, no credit card. If you
    contribute hardware, you get storage. That's the whole business
    model, which is to say there isn't one.

2.  **Reuse before recycle.** The most sustainable computer is the one
    that already exists. A laptop only goes to (responsible) recycling
    when it genuinely can't serve anymore.

3.  **Open and inspectable.** The stack is open source top to bottom —
    [Garage](https://garagehq.deuxfleurs.fr/) for storage,
    [Nebula](https://github.com/slackhq/nebula) for networking, and
    these docs for everything else. No black boxes.

4.  **Honest about limits.** Junk Net is built from old hardware on
    home internet connections. It will never out-perform a data centre,
    and we won't pretend otherwise. What it offers is durability,
    community ownership, and a price of zero.

## The stack, briefly

| Layer | Technology | Job |
|-------|-----------|-----|
| Storage | [Garage](https://garagehq.deuxfleurs.fr/) | Pools node disks into a replicated, S3-compatible object store |
| Network | [Nebula](https://github.com/slackhq/nebula) | Encrypted peer-to-peer mesh connecting nodes across homes and NATs |
| Hardware | Donated laptops | The junk. The whole point. |

Garage was built by [Deuxfleurs](https://deuxfleurs.fr/) specifically
for clusters of cheap, mismatched, unreliable machines connected over
ordinary internet links — which describes Junk Net exactly. Nebula
(from Slack's engineering team) handles the hard networking problems:
NAT traversal, certificates, and encrypted connectivity between
machines that live behind different home routers.

For the full picture, see [How it works](../how-it-works/index.md).

## Who's behind it

Junk Net is a community project by
[Aquainnis](https://aquainnis.com), starting in **Brisbane,
Australia**. It's currently in active development — see the
[roadmap](roadmap.md) for status, and the
[Brisbane pilot](../contribute/brisbane.md) page if you want in.

!!! question "Why the name?"

    Because it's a network made of junk. Some names just write
    themselves. (And yes, the tank-riding mascot energy is very much
    on purpose.)

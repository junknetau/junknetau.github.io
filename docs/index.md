---
title: Junk Net — the community cloud built from junk
description: >
  Junk Net turns donated old laptops into a free, community-owned,
  S3-compatible object store. Donate a machine, get storage.
  Starting in Brisbane, Australia.
hide:
  - navigation
  - toc
---

<div class="jn-hero" markdown>

<span class="jn-kicker">Starting in Brisbane · In development</span>

# Old laptops don't die.<br>They join the *Junk Net*.

<p class="jn-tagline">
Junk Net turns the laptops gathering dust in your drawer into a
free, community-owned cloud. Every donated machine becomes a storage
node in a mesh network. Donate a laptop, get free object storage.
No subscriptions. No lock-in. Just junk, working together.
</p>

[Donate a laptop](contribute/index.md){ .md-button .md-button--primary }
[See how it works](how-it-works/index.md){ .md-button }

</div>

<div class="jn-center" markdown>

## One person's junk is a whole community's cloud

</div>

<div class="grid cards" markdown>

-   :lucide-recycle:{ .lg .middle } __Rescue a laptop__

    ---

    That laptop in the drawer still works — it's just slow, old, or
    unloved. We wipe it, health-check it, and give it a new job:
    storage node.

    [:lucide-arrow-right: Donate a laptop](contribute/index.md)

-   :lucide-cloud:{ .lg .middle } __Get free storage__

    ---

    Contribute a machine and you get S3-compatible object storage in
    return. Back up your photos, host your files, keep your data in
    your own community.

    [:lucide-arrow-right: Use your storage](use/index.md)

-   :lucide-network:{ .lg .middle } __Join the mesh__

    ---

    Nodes connect over an encrypted [Nebula](https://github.com/slackhq/nebula)
    mesh and pool their disks with [Garage](https://garagehq.deuxfleurs.fr/),
    an object store built for exactly this kind of scrappy hardware.

    [:lucide-arrow-right: The architecture](how-it-works/index.md)

-   :lucide-heart-handshake:{ .lg .middle } __Own it together__

    ---

    Junk Net is free and community-owned. No venture capital, no
    "sunsetting your plan" emails. If the community runs it, the
    community keeps it.

    [:lucide-arrow-right: The cause](about/cause.md)

</div>

## The idea in 60 seconds

1.  **You donate an old laptop.** Anything reasonably modern that still
    powers on. We securely wipe it — every byte of your old data is
    destroyed.

2.  **It becomes a node.** We install the Junk Net OS image. The machine
    joins an encrypted mesh network and offers its disk to the cluster.

3.  **The cluster pools the storage.** Garage spreads three copies of
    every object across different machines in different homes, so no
    single dead laptop (or house fire) loses data.

4.  **You get free storage.** An S3-compatible endpoint and your own
    access keys. Works with the tools you already know — `rclone`,
    AWS CLI, Cyberduck, or any S3 library.

!!! note "Why *Tank Girl*?"

    Because the aesthetic fits. A tank bolted together from scrap,
    driven with attitude, that somehow works better than the shiny
    stuff. Junk Net is infrastructure with the same energy: built from
    what everyone else threw away, owned by nobody's shareholders.

## Why this matters

<div class="grid cards" markdown>

-   :lucide-leaf:{ .lg .middle } __Less e-waste__

    ---

    Making a new laptop emits hundreds of kilograms of CO₂ before it's
    ever switched on. The greenest computer is the one that already
    exists — so let's keep it working.

-   :lucide-piggy-bank:{ .lg .middle } __Free means free__

    ---

    Cloud storage prices only ever ratchet up. Junk Net storage is free
    for contributors, forever. The "payment" is the laptop you weren't
    using anyway.

-   :lucide-map-pin:{ .lg .middle } __Local & sovereign__

    ---

    Your data lives on machines in your own community — not in a
    hyperscaler's data centre on another continent, subject to someone
    else's terms of service.

</div>

## Starting in Brisbane :lucide-map-pin:

The first Junk Net cluster is being built in **Brisbane, Australia**.
If you're local and have a laptop to donate — or you want to host a
node at your place — we'd love to hear from you.

[Join the Brisbane pilot](contribute/brisbane.md){ .md-button .md-button--primary }

!!! warning "Junk Net is in development"

    This project is being actively built and the pilot cluster is not
    yet open for general use. Nothing here comes with an SLA — read
    [Trust & safety](how-it-works/trust.md) before storing anything you
    can't afford to lose. Follow the [roadmap](about/roadmap.md) to see
    where things are at.

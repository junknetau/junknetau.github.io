---
icon: lucide/circle-help
title: FAQ
description: Frequently asked questions about Junk Net.
---

# Frequently asked questions

## Donating

??? question "What kind of laptop can I donate?"

    Roughly: anything 64-bit that still powers on and has a working
    disk of 100&nbsp;GB or more. In practice that's most laptops made
    since about 2010. Cracked screens, dead batteries, and missing keys
    are all fine — a node doesn't need any of those things to serve
    storage. See [Donate a laptop](../contribute/index.md) for the
    full criteria.

??? question "What happens to my data on the laptop?"

    The entire disk is securely erased before anything else happens to
    the machine — every donated laptop is wiped before it's
    health-checked, imaged, or connected to anything. If you'd like to
    wipe it yourself first, even better; we'll wipe it again anyway.

??? question "What if my laptop is genuinely dead?"

    We'll take it off your hands and pass it to a certified e-waste
    recycler, and we may harvest working parts (RAM, disks) to upgrade
    other nodes. It won't go to landfill either way.

??? question "Do I have to keep hosting the laptop at my house?"

    No. There are two ways to contribute: **donate** a machine (we
    house it) or **host** a node (it lives at your place, plugged into
    your power and internet). Hosting is what makes the network truly
    distributed, but it's entirely optional. See the
    [Brisbane pilot](../contribute/brisbane.md) page for what hosting
    involves.

## Using the storage

??? question "How much storage do I get?"

    During the pilot, allocations are set per-contributor based on
    what the cluster can sustain — as a rule of thumb, expect an
    allocation in proportion to the usable capacity you contributed.
    The exact policy will be settled (publicly) as the pilot runs.
    Remember every byte you store is stored three times across the
    cluster, so the cluster's usable capacity is roughly a third of its
    raw capacity.

??? question "Is it fast?"

    It's honest-speed. Nodes live on home internet connections, and
    Australian upload bandwidth is what it is. Junk Net is great for
    backups, archives, photo libraries, and file sharing — things where
    durability matters more than latency. It is not a CDN and it won't
    stream 4K video to a crowd.

??? question "Can I lose data?"

    Every object is stored as **three copies on three different
    machines in different homes**, so single failures — a dead disk, a
    power cut, an unplugged node — don't cause loss. But Junk Net is a
    community project in development, not a commercial service with an
    SLA. Treat it as one copy in your backup strategy, not the only
    copy. See [Trust & safety](../how-it-works/trust.md).

??? question "Can the person hosting a node read my files?"

    Not meaningfully. Objects are split into chunks that are scattered
    across many machines, so any single node holds an incomplete
    jumble of compressed fragments. That said, Garage does not encrypt
    data at rest by default, so for anything sensitive we recommend
    (and document) client-side encryption — then nobody but you can
    read your data, ever. Details in
    [Trust & safety](../how-it-works/trust.md).

??? question "What tools work with it?"

    Anything that speaks S3: `rclone`, the AWS CLI, `s3cmd`, Cyberduck,
    restic, Duplicati, and every major language's S3 SDK. See
    [Using your storage](../use/index.md) for quickstarts.

## The project

??? question "Who runs Junk Net?"

    It's a community project by [Aquainnis](https://aquainnis.com),
    starting in Brisbane, Australia. During the pilot, cluster
    administration (the mesh certificate authority and the Garage
    admin credentials) is held by the project maintainer. Long-term,
    the goal is community governance — that transition is on the
    [roadmap](roadmap.md).

??? question "What does it cost?"

    Nothing. The entry fee is a laptop you weren't using. Hosts pay
    for the small amount of power their node draws (roughly
    AU$30–80/year for a typical old laptop) — that's the only ongoing
    cost anywhere in the system, and it's borne voluntarily.

??? question "What stops someone storing something illegal?"

    Access is per-community and per-person — keys are issued to known
    contributors, not anonymous signups, and the
    [trust & safety](../how-it-works/trust.md) page sets out the
    acceptable-use expectations and what happens when they're broken.
    Chunking means no host ever holds a stranger's complete files, and
    client-side encryption means hosts can't be expected to police
    content they cannot see — the accountability sits with the person
    who holds the keys.

??? question "Is this like BitTorrent / IPFS / Filecoin?"

    Cousins, not siblings. Junk Net isn't a global anonymous network
    and there's no token. It's a *local* cluster run by people who know
    each other, using boring, proven tools (Garage, Nebula, systemd).
    Community-scale trust is the feature.

??? question "Why Garage and not MinIO or Ceph?"

    MinIO and Ceph assume data-centre conditions: uniform hardware,
    fast low-latency links, disks that mostly don't disappear. Garage
    was designed by [Deuxfleurs](https://deuxfleurs.fr/) for the
    opposite: mismatched consumer machines in people's homes on
    ordinary internet connections. That's Junk Net's exact situation.
    More in [The storage layer](../how-it-works/storage.md).

??? question "Can I leave? What happens to my node and my data?"

    Yes, any time. Download your data (no egress fees, obviously),
    and if you're hosting a node we'll drain it — Garage rebalances its
    chunks onto other machines — and you can keep, return, or recycle
    the hardware. No lock-in is the point.

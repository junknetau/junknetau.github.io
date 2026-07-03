---
icon: lucide/shield
title: Trust & safety
description: >
  What node hosts can and can't see, how to make your data unreadable
  to everyone but you, and who holds the keys.
---

# Trust & safety

Your files, physically, will sit on old laptops in other people's
houses. That sentence should raise questions. This page answers them
plainly — including the uncomfortable parts.

## What a node host can (and can't) see

Someone hosting a Junk Net node has root on that machine, so assume
they can inspect anything stored on it. Here's what that actually gets
them:

- **Fragments, not files.** Garage splits every object into ~1&nbsp;MiB
  blocks, compresses them, and scatters the copies across the cluster
  by hash. A single node holds a jumble of anonymous compressed chunks
  from many users — no filenames, no directory structure, no complete
  objects of any size beyond one block.
- **No user metadata.** Bucket names, object keys, and access keys live
  in the replicated metadata store; a host poking at their own disk
  sees block hashes, not your photo album's name. (A host whose node
  carries metadata replicas can see more — which is one more reason
  for the next section.)
- **Nothing in transit.** All inter-node traffic runs inside the
  Nebula mesh: end-to-end encrypted, mutually authenticated. Nobody's
  ISP, router, or nosy flatmate sees cluster traffic in the clear.

That's meaningful protection against casual snooping. It is **not**
cryptographic protection: Garage does not encrypt blocks at rest, and a
sufficiently determined and technical host could reassemble whatever
fragments their disk happens to hold. So:

## The house rule: encrypt before you upload

For anything you wouldn't pin to a community noticeboard, encrypt
client-side. Then hosts hold ciphertext, the maintainer holds
ciphertext, a burglar who steals a node holds ciphertext — and only you
hold the key.

The [usage guide](../use/index.md) documents this as the default path
with `rclone crypt`, which encrypts file contents *and* names before
anything leaves your machine:

``` ini title="~/.config/rclone/rclone.conf (sketch)"
[junknet]
type = s3
provider = Other
endpoint = https://s3.junknet.au   # pilot endpoints handed out directly

[junknet-crypt]
type = crypt
remote = junknet:my-bucket
password = <yours, generated, backed up — we can never recover it>
filename_encryption = standard
```

!!! warning "The trade"

    Client-side encryption means **we cannot help you recover your
    data if you lose your password.** That's not a flaw — it's the
    entire point. Back your key up somewhere that isn't Junk Net.

Backup tools like **restic** bring their own encryption and work
beautifully over S3 — using them against Junk Net gives you encrypted,
deduplicated, versioned backups with zero extra ceremony.

## Who holds which keys

Trust models are about keys, so here's the honest ledger during the
pilot:

| Key / credential | Held by | What it controls |
|------------------|---------|------------------|
| Your S3 access keys | You | Read/write to your buckets |
| Your encryption password | **Only you** | Whether anyone can ever read your data |
| Garage RPC secret | Every node (config file) | Whether a machine may join cluster RPC |
| Garage admin token | Project maintainer | Creating keys/buckets, cluster admin |
| Nebula CA key | Project maintainer (offline) | Who may join the mesh at all |

Yes: during the pilot, one person (the maintainer) can admit nodes and
issue credentials. That's the honest centralisation of a project at
this stage, and dissolving it — shared custody of the CA and admin
credentials, written governance — is an explicit
[roadmap](../about/roadmap.md) phase, not an afterthought.

## Abuse: the community's problem, handled by design

**Access is not anonymous.** Keys are issued to known contributors —
people who handed over a laptop or host a node. This is a community
storage co-op, not a public dropbox, and that's the single strongest
abuse control available.

**Hosts are shielded by architecture.** Chunking means no host ever
holds a stranger's complete files; client-side encryption means hosts
*cannot* inspect content even in principle. Responsibility sits where
the keys sit: with the account holder.

**The rules are boring and firm.** Storing material that's illegal to
possess, using the cluster to serve piracy, or attacking the
infrastructure gets your keys revoked and, where the law requires it,
reported. A written acceptable-use policy agreed by members is part of
the governance milestone.

## What Junk Net does *not* promise

In development, community-run, built from junk — so, plainly:

- **No SLA.** The cluster may be slow, briefly unavailable, or down
  for maintenance without notice.
- **No guarantee against loss.** Three zone-separated replicas make
  loss genuinely unlikely, but "unlikely" is not "impossible", and
  pilot-phase software gets reconfigured. Junk Net should be *one* of
  your copies — it pairs beautifully with a backup strategy and makes
  a terrible only-copy.
- **No warranty, no liability.** It's free, it's best-effort, and it's
  honest about both.

What it *does* promise: your donated laptop's old data is destroyed
before the machine touches the network, your stored data is never
sold, mined, or advertised against, and you can take everything and
leave at any time — no fees, no dark patterns, no hostage negotiations.

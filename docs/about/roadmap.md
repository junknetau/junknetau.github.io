---
icon: lucide/map
title: Roadmap
description: Where Junk Net is at, and where it's going.
---

# Roadmap

Junk Net is in active development. This page is the honest state of
things — expect it to change as reality continues to set in.

## Phase 0 — Design & docs :lucide-check:

*Mostly done.*

- [x] Choose the storage layer ([Garage](../how-it-works/storage.md))
- [x] Choose the mesh layer ([Nebula](../how-it-works/mesh.md))
- [x] Prove out a hardened single-node Garage service
      ([guide](../operators/single-node.md))
- [x] Document the architecture and the operator playbooks
- [x] Publish this site

## Phase 1 — Brisbane pilot :lucide-hammer:

*In progress. The current focus.*

- [ ] Collect the first batch of donated laptops
- [ ] Wipe, health-check, and hand-provision 3–5 nodes
- [ ] Stand up the Nebula lighthouse and certificate authority
- [ ] Form the first replicated cluster (`replication_factor = 3`)
      across at least two homes
- [ ] Issue the first contributor access keys
- [ ] Run it for real: measure durability, node churn, and bandwidth
      over a few months of actual use

## Phase 2 — The node image :lucide-disc-3:

*The magic trick: a laptop becomes a node by booting a USB stick.*

- [ ] Automated installer image (Debian-based) — wipe, partition,
      install, enroll
- [ ] First-boot provisioning: generate keys, request a signed Nebula
      certificate, register with the cluster
- [ ] Sensible laptop defaults baked in: ignore lid-close, auto-restart
      services, reconnect on network drop
- [ ] Node health telemetry: SMART monitoring and alerts *before*
      disks die, not after
- [ ] A simple "is my node okay?" status page for hosts

## Phase 3 — Growing the mesh :lucide-network:

- [ ] Nodes in enough different homes that zone-aware replication is
      meaningfully geographic
- [ ] Public S3 endpoint with TLS (`s3.junknet.au`)
- [ ] Client-side encryption documented as the default path, not an
      option
- [ ] Capacity/allocation policy settled and published
- [ ] Second community cluster outside the pilot group

## Phase 4 — Community governance :lucide-users:

*The part that makes "community-owned" true rather than aspirational.*

- [ ] Shared custody of the mesh CA and Garage admin credentials
- [ ] Written acceptable-use and abuse-handling policy, agreed by
      members
- [ ] A lightweight membership process for new communities that want
      to start their own Junk Net
- [ ] Succession plan: the network must survive any single person
      losing interest, including the founder

!!! note "Want to move something up this list?"

    The fastest way is to show up with a laptop.
    [Join the Brisbane pilot](../contribute/brisbane.md).

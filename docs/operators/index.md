---
icon: lucide/wrench
title: Running a node
description: >
  The operator playbooks — how a Junk Net node is built, joined to the
  mesh, and added to the cluster.
---

# Running a node

These are the playbooks used to build Junk Net nodes by hand during the
pilot. They double as the specification for the automated node image
(roadmap Phase 2) — everything the image will do, you can do yourself
with these pages and a terminal.

They're written to be followed by anyone comfortable on a Linux command
line. If that's you and you'd like to help run the pilot cluster, see
[getting involved](../contribute/brisbane.md).

## Anatomy of a node

Every Junk Net node is the same small stack on a donated laptop:

| Component | Role |
|-----------|------|
| Debian (or similar) minimal install | Boring, stable base OS |
| `garage` (systemd service, hardened) | Serves this node's disk to the cluster |
| `nebula` (systemd service) | Connects the node to the encrypted mesh |
| SMART monitoring | Warns us *before* the old disk dies |

Plus laptop-specific tuning: lid-close does nothing, services restart
on failure, the machine returns after power loss, and the battery gets
treated as the free UPS it is.

## The playbooks, in order

<div class="grid cards" markdown>

-   :lucide-server:{ .lg .middle } __1. Single node__

    ---

    Install Garage as a hardened systemd service on one machine, with
    a dedicated user and locked-down permissions. The foundation for
    everything else.

    [:lucide-arrow-right: Garage as a Linux service](single-node.md)

-   :lucide-network:{ .lg .middle } __2. Join the mesh__

    ---

    Install Nebula, get a certificate signed, and give the node its
    permanent mesh address.

    [:lucide-arrow-right: Set up the mesh](nebula.md)

-   :lucide-boxes:{ .lg .middle } __3. Join the cluster__

    ---

    Connect the node to its peers, assign it a zone and weight in the
    cluster layout, and watch the data flow in.

    [:lucide-arrow-right: Cluster operations](cluster.md)

</div>

## Laptop tuning cheat-sheet

The settings every node needs that a server guide won't mention:

``` bash
# Closing the lid must NOT suspend the node
sudo mkdir -p /etc/systemd/logind.conf.d
sudo tee /etc/systemd/logind.conf.d/junknet.conf > /dev/null <<'EOF'
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
EOF
sudo systemctl restart systemd-logind
```

``` bash
# Keep an eye on the elderly disk: install and enable SMART monitoring
sudo apt install smartmontools
sudo systemctl enable --now smartmontools

# Quick health read — check this BEFORE trusting a donated disk
sudo smartctl -H /dev/sda
sudo smartctl -A /dev/sda | grep -Ei 'reallocated|pending|uncorrect'
```

Also worth doing where hardware allows: set "power on after AC restore"
in the BIOS/UEFI, and disable OS sleep/hibernate entirely
(`sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target`).

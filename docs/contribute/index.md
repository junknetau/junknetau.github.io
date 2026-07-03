---
icon: lucide/laptop
title: Donate a laptop
description: >
  What laptops Junk Net can use, what happens to your machine and your
  old data, and what you get in return.
---

# Donate a laptop

That laptop in the drawer? This is its redemption arc.

## What we can use

The bar is deliberately low — the whole point is machines nobody wants:

| | Minimum | Notes |
|---|---------|-------|
| **CPU** | 64-bit (x86_64) | Most laptops since ~2010. ARM support is on the wishlist |
| **RAM** | 4 GB | Garage is light; 2 GB machines may still work as small nodes |
| **Disk** | 100 GB, healthy | The disk *is* the donation — bigger is better. SSDs and spinners both welcome |
| **Condition** | Powers on | Cracked screen? Dead battery? Missing keys? All fine — nodes don't need them |
| **Network** | Ethernet port or working wifi | Ethernet preferred for hosted nodes |

!!! tip "Don't pre-judge your junk"

    Bring it anyway. If it can't serve as a node, we'll harvest working
    parts (RAM and disks upgrade other nodes) and pass the rest to a
    certified e-waste recycler. Either way it stays out of landfill,
    which is half the mission.

**What we can't take (yet):** desktops and servers are technically fine
but bulky to house, phones and tablets aren't supported, and anything
water-damaged goes straight to recycling.

## What happens to your machine

1.  **Secure wipe — first, always.** The entire disk is erased before
    the machine is even health-checked. Every byte of your old life on
    that laptop is destroyed. Want to wipe it yourself first? Great —
    we'll still wipe it again.

2.  **Health check.** Disk SMART data, memory test, thermal sanity.
    Machines that pass become nodes; machines that don't become spare
    parts and responsibly-recycled scrap.

3.  **Imaging.** We install the Junk Net node image: Debian, Garage,
    Nebula, and laptop-appropriate settings (lid stays closed, services
    auto-restart, disk health is monitored so we hear a disk *dying*
    rather than discover it dead).

4.  **Enrollment.** The node gets a signed mesh certificate and joins
    the cluster. Garage rebalances, and your old laptop starts holding
    tiny encrypted-in-transit fragments of the community's data.

## What you get

- **Free S3-compatible storage**, allocated in proportion to the usable
  capacity you contributed (exact policy is being settled during the
  pilot — see the [FAQ](../about/faq.md)).
- **Access keys and a quickstart** — you'll be uploading within
  minutes of getting them. See [Using your storage](../use/index.md).
- **The moral high ground** at dinner parties, when the topic of cloud
  subscription pricing comes up.

## Two ways to contribute

=== "Donate"

    Hand the laptop over and you're done — we house it, power it, and
    run it. Perfect if you just want the drawer back and the storage.

=== "Host"

    The laptop (yours or an assigned one) lives at your place, plugged
    into power and your internet. This is what makes Junk Net
    genuinely distributed — every hosting household is a *zone*, and
    more zones means every object's three copies are spread across
    more of the city.

    What hosting asks of you:

    - a power point (nodes draw 10–25 W — roughly AU$30–80/year),
    - a spot near your router (ethernet ideally, solid wifi is fine),
    - and tolerance for one quietly humming laptop that you agree not
      to unplug on a whim. (Unplugging *with notice* is fine — we
      drain the node first and nothing is lost.)

Ready?

[Join the Brisbane pilot](brisbane.md){ .md-button .md-button--primary }

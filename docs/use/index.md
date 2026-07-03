---
icon: lucide/cloud
title: Using your storage
description: >
  Quickstart for Junk Net's S3-compatible storage — rclone, AWS CLI,
  Cyberduck, and client-side encryption.
---

# Using your storage

Junk Net speaks the **S3 API**, so it works with tools you may already
use. This page takes you from access keys to encrypted backups.

!!! info "Pilot phase"

    The public endpoint `s3.junknet.au` is not live yet. During the
    [Brisbane pilot](../contribute/brisbane.md), contributors receive
    their endpoint address together with their access keys — substitute
    it wherever this page says `<endpoint>`.

## What you receive

When your contribution is set up, you get three things:

| Item | Looks like | Treat it as |
|------|-----------|-------------|
| Endpoint | `https://<endpoint>` | Public knowledge |
| Key ID | `GK31c2f218a2e44f485b94239e` | Semi-secret |
| Secret key | `b892c0665f0ada8a4755dae98baa3b133590e11dae3bcc1f9d769d67f16c3835` | **A password. Store it in a password manager.** |

The S3 *region* for all Junk Net clusters is `garage`.

## Quickstart

=== "rclone (recommended)"

    [rclone](https://rclone.org/) is the Swiss Army knife of cloud
    storage, and its `crypt` layer makes client-side encryption easy —
    which is why it's the recommended client.

    Add to `~/.config/rclone/rclone.conf`:

    ``` ini
    [junknet]
    type = s3
    provider = Other
    endpoint = https://<endpoint>
    region = garage
    access_key_id = <your-key-id>
    secret_access_key = <your-secret-key>
    ```

    Then:

    ``` bash
    rclone mkdir junknet:my-bucket        # create a bucket
    rclone copy ~/Photos junknet:my-bucket/photos   # upload
    rclone ls junknet:my-bucket           # list
    rclone copy junknet:my-bucket/photos ~/Restored # download
    ```

=== "AWS CLI"

    ``` bash
    aws configure --profile junknet
    # AWS Access Key ID:     <your-key-id>
    # AWS Secret Access Key: <your-secret-key>
    # Default region name:   garage
    ```

    Garage needs the endpoint passed explicitly:

    ``` bash
    aws --profile junknet --endpoint-url https://<endpoint> \
        s3 mb s3://my-bucket
    aws --profile junknet --endpoint-url https://<endpoint> \
        s3 cp ~/Photos s3://my-bucket/photos --recursive
    aws --profile junknet --endpoint-url https://<endpoint> \
        s3 ls s3://my-bucket
    ```

=== "Cyberduck (GUI)"

    For a point-and-click experience on macOS/Windows:

    1. **Open Connection** → choose **Amazon S3**.
    2. Server: `<endpoint>` (port 443).
    3. Access Key ID / Secret: your keys.
    4. Connect — buckets appear as folders; drag and drop away.

=== "Code (Python)"

    Any S3 SDK works. With `boto3`:

    ``` python
    import boto3

    s3 = boto3.client(
        "s3",
        endpoint_url="https://<endpoint>",
        region_name="garage",
        aws_access_key_id="<your-key-id>",
        aws_secret_access_key="<your-secret-key>",
    )

    s3.upload_file("holiday.zip", "my-bucket", "holiday.zip")
    print(s3.list_objects_v2(Bucket="my-bucket")["KeyCount"])
    ```

## Encrypt before you upload (please)

Your data will physically live on laptops in other people's homes.
It's chunked and scattered — no host holds your complete files — but it
is **not encrypted at rest** unless you encrypt it. The
[trust model](../how-it-works/trust.md) explains why the house rule is:

> If you wouldn't pin it to a community noticeboard, encrypt it
> client-side.

With rclone, add a `crypt` remote on top of your bucket:

``` bash
rclone config
# n) New remote → name: junknet-crypt → type: crypt
# remote: junknet:my-bucket
# filename_encryption: standard
# password: generate one, and BACK IT UP outside Junk Net
```

Then use `junknet-crypt:` exactly like before — contents *and*
filenames are encrypted on your machine before upload:

``` bash
rclone copy ~/Documents/tax junknet-crypt:tax
```

!!! danger "Your password is unrecoverable — by design"

    Nobody in Junk Net can reset or recover your encryption password.
    Lose it and the data is ciphertext forever. Store it in a password
    manager, and keep a copy somewhere that isn't Junk Net.

## Backups: the killer app

Junk Net's sweet spot is durable, boring, free backup — and
[restic](https://restic.net/) does encrypted, deduplicated, versioned
backups over S3 natively:

``` bash
export AWS_ACCESS_KEY_ID=<your-key-id>
export AWS_SECRET_ACCESS_KEY=<your-secret-key>

restic -r s3:https://<endpoint>/my-backups init
restic -r s3:https://<endpoint>/my-backups backup ~/Documents
restic -r s3:https://<endpoint>/my-backups snapshots
```

Restic encrypts everything client-side by default, so the house rule is
satisfied automatically.

## Fair use & expectations

- **Store within your allocation.** Every byte you store occupies three
  disks across the community — allocations exist so the maths keeps
  working.
- **It's honest-speed.** Nodes live on residential connections. Bulk
  uploads and restores work fine; latency-sensitive workloads and
  public high-traffic hosting are the wrong tool for this network.
- **No SLA during the pilot.** Keep another copy of anything you truly
  can't lose — Junk Net makes a great *second* copy and a bad *only*
  copy.
- **The rules:** nothing illegal to possess, no serving piracy, no
  attacking the infrastructure. Keys are issued to known community
  members and can be revoked. Boring rules, firmly held.

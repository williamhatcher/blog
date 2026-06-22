---
title: Making Bootable Cisco ISO Images
date: 2026-06-19 21:07 -0500
tags:
  - Cisco
  - ISO
description: Using open source tools lets us create a bootable ISO when Cisco won't give us one.
---

The Cisco Unified Communications Manager ISO on the Cisco Downloads Portal is not bootable; to get a bootable ISO you have to reach out to TAC. Here's how to make it bootable.

> Note: This has been tested on UCM 15.0.1 and IM&P 15.0.1.

1. Create some tempoary directories to mount the ISO:
`mkdir -p /tmp/cucm/iso_mount /tmp/cucm/iso_extract`
2. Mount the ISO:
`sudo mount -o loop,ro <ISO> /tmp/cucm/iso_mount`
3. Copy the contents over so we can modify them:
`rsync -a /tmp/cucm/iso_mount/ /tmp/cucm/iso_extract/ && cd /tmp/cucm/iso_extract`
4. (Bonus) Modify the hardware scripts to allow running on KVM:
Edit `Cisco/install/conf/callmanager_product.conf`

On the line containing `NOT,       *,*,*`... replace NOT with `VAL`.

Note: IM&P uses `cups_product.conf` and has a sneaky NOT * a few lines above VAL * that needs replacing.

5. Recreate the ISO making it bootable:
```sh
genisoimage -r -J -joliet-long -V "CDROM" -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -output ~/OUTPUT.iso /tmp/cucm/iso_extract/
```
> Note: The volume must be "CDROM" as the installer looks for the files in `/dev/disk/by-label/CDROM`.

6. Recalculate the md5 checksum:
```sh
implantisomd5 --force ~/OUTPUT.iso
```

Now you have an ISO that can be booted and installed on any platform.

I've noticed that the installer will consume at least 20gig of RAM if given; I think there might be a memory leak...

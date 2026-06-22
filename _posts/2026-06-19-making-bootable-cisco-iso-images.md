---
title: Making Bootable Cisco ISO Images
date: 2026-06-19 21:07 -0500
last_modified_at: 2026-06-19 13:00 -0500
tags:
  - Cisco
  - ISO
  - Unified Communications
  - Home Lab
description: Using open source tools lets us create a bootable ISO when Cisco won't give us one.
---

The Cisco Unified Communications Manager ISO on the Cisco Downloads Portal is not bootable; to get a bootable ISO you have to reach out to TAC. Here's how to make it, and other Cisco ISOs bootable by yourself.

> This has been tested on version 15SU4 of Communications Manager (CallManager), IM & Presence, and Unity Connect. It _should_ work for other Linux-based Cisco ISOs.
{: .prompt-tip }

> This is not suitable for production! Use at your own risk. I am using this in my home lab where the stakes are low.
{: .prompt-warning }


1. Create some tempoary directories to mount the ISO:
`mkdir -p /tmp/cucm/iso_mount /tmp/cucm/iso_extract`

2. Mount the ISO:
`sudo mount -o loop,ro <ISO> /tmp/cucm/iso_mount`

3. Copy the contents over so we can modify them:
```sh
rsync -a /tmp/cucm/iso_mount/ /tmp/cucm/iso_extract/ && cd /tmp/cucm/iso_extract
```

4. (Optional) Modify the hardware scripts to allow running on KVM (only on 15SU4 and later):
```sh
sed -i '/<server_models>/a \           VAL,        *,     *,      *,    *,      *,     *,      *,    *,      *' *product.conf
```
> Explanation:
This adds a valid hardware entry that matches any configuraton at the top of the list bypassing all other checks.

5. Recreate the ISO making it bootable:
```sh
genisoimage -r -J -joliet-long -V "CDROM" -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -output ~/OUTPUT.iso /tmp/cucm/iso_extract/
```
> Explanation:
`-r -J -joliet-long` add Joliet and Rock Ridge extensions which use Unicode and allow for long file names.
<br>
`-V "CDROM"` sets the volume to "CDROM" as the installer looks for the files in `/dev/disk/by-label/CDROM:/`
<br>
See `man genisoimage`

6. Recalculate the md5 checksum:
```sh
implantisomd5 --force ~/OUTPUT.iso
```

7. Unmount and remove extracted files:
`sudo umount /tmp/cucm/iso_mount ; rm -rf /tmp/cucm/iso_extract/`

Now you have an ISO that can be booted and installed on any platform.


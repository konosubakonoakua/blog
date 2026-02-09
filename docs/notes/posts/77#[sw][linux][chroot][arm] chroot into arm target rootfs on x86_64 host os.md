---
number: 77
title: "[sw][linux][chroot][arm] chroot into arm target rootfs on x86_64 host os"
state: open
url: https://github.com/konosubakonoakua/blog/issues/77
author: konosubakonoakua
createdAt: 2024-08-03T17:12:35Z
updatedAt: 2024-09-25T22:52:27Z
labels: ["Linux"]
---

# 77 [sw][linux][chroot][arm] chroot into arm target rootfs on x86_64 host os

```bash
sudo apt install qemu-user-static
# plugin U disk
sudo cp $(which qemu-user-static) /media/rf/rootfs/usr/bin
sudo chroot /media/rf/rootfs qemu-arm-static /bin/bash

# should enter target rootfs
qemu-arm-static /bin/ls
qemu-arm-static /bin/vi
```

---
number: 103
title: "[sw][vm][linux] virtualbox shared folder on windows"
state: open
url: https://github.com/konosubakonoakua/blog/issues/103
author: konosubakonoakua
createdAt: 2024-09-30T07:45:04Z
updatedAt: 2024-10-04T18:59:06Z
labels: []
---

# 103 [sw][vm][linux] virtualbox shared folder on windows

## Setup shared folder on host
<img src="https://github.com/user-attachments/assets/fd2a7f25-5b51-4725-929c-3e37cb639eb1" width="50%" />

## Add user to `vboxsf` group
```bash
sudo usermod -aG vboxsf $(whoami)
newgrp vboxsf
```
## reboot

---

## Comments

### konosubakonoakua · 2024-10-04T18:59:05Z

## vmware
> [!TIP]
> In case you also need to kown how to do it in vmware.
> https://github.com/konosubakonoakua/blog/issues/106#issuecomment-2394337749

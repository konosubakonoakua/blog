---
number: 191
title: "[vm] virtualbox setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/191
author: konosubakonoakua
createdAt: 2025-07-11T06:50:54Z
updatedAt: 2025-07-11T06:50:58Z
labels: []
---

# 191 [vm] virtualbox setup

## setup
```powershell
winget install Oracle.VirtualBox
```
## issues
### Can not import `.vbox` file: `E_INVALIDARG (0x80070057)`
- restart vbox manager with admin rights
### only super user can access shared folder
- `sudo usermod -aG vboxsf $(whoami)`
- `reboot`

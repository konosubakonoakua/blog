---
number: 102
title: "[sw][linux][ubuntu] remote desktop on ubuntu"
state: open
url: https://github.com/konosubakonoakua/blog/issues/102
author: konosubakonoakua
createdAt: 2024-09-30T04:48:30Z
updatedAt: 2024-09-30T05:20:17Z
labels: []
---

# 102 [sw][linux][ubuntu] remote desktop on ubuntu

## Turn on Sharing
<img src="https://github.com/user-attachments/assets/1dec60ee-39b4-47d4-9235-2c8fa60df787" width="70%" />

## Allow Locked Remote Desktop
> [!IMPORTANT]
> If you skip this step, you cannot use rdp without login..., It's a dead loop.
> Also, this plugin don't work until you login, In another word, can't reboot then login via rdp.
> If you want to use rdp after boot, I recommend that login via `RustDesktop`, then use RDP. Or just use RustDesktop.

```bash
sudo apt install gnome-shell-extension-manager
```

| <img src="https://github.com/user-attachments/assets/ac9a1cfd-a198-4821-ad97-4d115fa79ef7" width="100%" /> | <img src="https://github.com/user-attachments/assets/fe3a8d7a-726d-4773-a161-fff17bcc60df" width="100%"/> |
|-|-|

---

## Comments

### konosubakonoakua · 2024-09-30T05:20:16Z

## references
- https://cn.linux-console.net/?p=8673
- https://losst.pro/en/how-to-enable-remote-desktop-in-ubuntu-22-04-23-10

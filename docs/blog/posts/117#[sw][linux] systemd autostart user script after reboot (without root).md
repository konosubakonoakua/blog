---
number: 117
title: "[sw][linux] systemd autostart user script after reboot (without root)"
state: open
url: https://github.com/konosubakonoakua/blog/issues/117
author: konosubakonoakua
createdAt: 2024-10-23T07:24:03Z
updatedAt: 2024-10-23T07:24:10Z
labels: []
---

# 117 [sw][linux] systemd autostart user script after reboot (without root)

```bash
mkdir -p $HOME/.config/systemd/user/
cat > $HOME/.config/systemd/user/vmhgfs-fuse.service << EOF
[Unit]
Description=VMware Hgfs Shared Folder Service
Requires=vmhgfs-fuse.service
After=vmhgfs-fuse.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/$USER
ExecStart=vmhgfs-fuse .host:/ /mnt/winshared/ -o subtype=vmhgfs-fuse
ExecStop=umount /mnt/winshared
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
EOF

```

```bash
systemctl --user start vmhgfs-fuse.service
systemctl --user enable vmhgfs-fuse.service
```

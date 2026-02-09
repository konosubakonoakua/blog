---
number: 82
title: "[sw][linux] linux/windows samba file share"
state: open
url: https://github.com/konosubakonoakua/blog/issues/82
author: konosubakonoakua
createdAt: 2024-09-19T18:04:10Z
updatedAt: 2025-12-17T08:19:55Z
labels: ["Linux"]
---

# 82 [sw][linux] linux/windows samba file share

## Step 1 - Linux smb server
```bash
# install samba deps
sudo apt install samba smbclient
```
```bash
# enable & start samba sersvice
sudo systemctl start smbd.service
sudo systemctl enable smbd.service
sudo systemctl start nmbd.service
sudo systemctl enable nmbd.service
```
```bash
# add user to group
sudo usermod -a -G sambashare $USER
```
```bash
# set user password
sudo smbpasswd -a $USER
```
```bash
# config share path
# net usershare add <sharename> <sharepath> "<sharedesc>" Everyone:F guest_ok=n
net usershare add share $HOME "home" Everyone:F guest_ok=n
```
```yaml
## /etc/samba/smb.conf
[share]
    path = /root
    writable = yes
    browsable = yes
    public = yes
    create mask = 0777
    directory mask = 0777
    force user = root
    max connections = 10

[share]
    path = /boot
    writable = yes
    browsable = yes
    public = yes
    create mask = 0777
    directory mask = 0777
    force user = root
    max connections = 10
```
```bash
# if you want to access softlink, use `mount --bind` or `fstab` instead
cd /root
mkdir boot
mount --bind /boot /root/boot
```
```bash
# fstab
/some/real/path  /srv/samba/share/mybind  none  bind  0  0
```
```bash
# test
smbclient //127.0.0.1/<sharename> -U <username>%<passwd>
```
## Step 2 - Windows Client
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -All
```
```bash
ping <your linux pc ip address>
```
- type `\\<your linux ip>\<sharename>` in ` windows explorer`.
- map a network disk by `net use Z: \\<server>\<shared folder> /user:<name> <pswd>`
- use `net use` in cmd to list all connections

---

## Comments

### konosubakonoakua · 2024-09-23T16:58:09Z

## linux mount windows shared
### windows side
[在 Windows 中通过网络共享文件](https://support.microsoft.com/zh-cn/windows/%E5%9C%A8-windows-%E4%B8%AD%E9%80%9A%E8%BF%87%E7%BD%91%E7%BB%9C%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6-b58704b2-f53a-4b82-7bc1-80f9994725bf)
### linux side
You may encounter `mount read-only` issue if cifs-utils is not installed.
```bash
sudo apt-get install cifs-utils
```

```bash
# create mount point
sudo mkdir -p /mnt/winshare
```
```bash
# list all sharenames
smbclient -L //10.9.8.7 -U user%password
```
```bash
sudo mount -t cifs //10.9.8.7/<sharename?> /mnt/winshare -o username=?,password=?
```
e.g.
```bash
sudo mount -t cifs //rf-pc-shiro.local/D/tools/petalinux /tools/petalinux/ -o username=RF-PC-SHIRO
```

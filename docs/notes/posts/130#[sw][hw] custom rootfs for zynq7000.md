---
number: 130
title: "[sw][hw] custom rootfs for zynq7000"
state: open
url: https://github.com/konosubakonoakua/blog/issues/130
author: konosubakonoakua
createdAt: 2024-12-12T07:56:16Z
updatedAt: 2025-05-28T07:18:05Z
labels: []
---

# 130 [sw][hw] custom rootfs for zynq7000

## rootfs from eewiki
- https://rcn-ee.com/rootfs/eewiki/minfs/
  - https://forum.digikey.com/t/debian-getting-started-with-the-zynq-7000/14380

---

## Comments

### konosubakonoakua · 2025-03-31T02:24:06Z

## root passwd reset (`/etc/shadow`)
```bash
openssl passwd -1 "your-password-here"
```
copy generated password to `/etc/shadow`:
```
# cat /etc/shadow
root:<your password>:10933:0:99999:7:::
```


### konosubakonoakua · 2025-03-31T03:10:39Z

https://github.com/Xilinx/PYNQ/tree/master/sdbuild/scripts

### konosubakonoakua · 2025-03-31T03:28:13Z

## deps

### debootstrap
```bash
sudo apt install qemu-user-static debootstrap binfmt-support
```
if debootstrap version is too low:
```bash
git clone https://salsa.debian.org/installer-team/debootstrap.git
cd debootstrap
sudo make install
```
Or download form ftp site: https://ftp.debian.org/debian/pool/main/d/debootstrap/
```bash
wget https://ftp.debian.org/debian/pool/main/d/debootstrap/debootstrap_1.0.140_all.deb
```

### konosubakonoakua · 2025-03-31T03:47:56Z

## References

<details>

- https://github.com/ikwzm/ZynqMP-FPGA-Debian12/blob/v7.0.0/doc/build/debian12-rootfs.md
- https://github.com/PyHDI/zynq-linux
- https://wiki.debian.org/InstallingDebianOn/Xilinx/ZC702/wheezy
- https://forum.digikey.com/t/debian-getting-started-with-the-zynq-7000/14380
- https://blog.night-shade.org.uk/2013/12/building-a-pure-debian-armhf-rootfs/
- https://www.slideshare.net/shtaxxx/debian-linux-on-zynq-setup-flow-vivado-20154
- https://fpgacpu.wordpress.com/2013/05/24/yet-another-guide-to-running-linaro-ubuntu-desktop-on-xilinx-zynq-on-the-zedboard/
- http://lucasbrasilino.com/posts/Custom-debian-for-zynq-ultrascale+-mpsoc-platform/
- https://www.ashtonjohnson.com/2019/10/ubuntu-nfs-rootfs-on-zynq-using.html
- https://doc.embedfire.com/linux/stm32mp1/quick_start_guide/zh/latest/quick_start/board_startup/board_startup.html
- https://akhileshmoghe.github.io/_post/linux/debian_minimal_rootfs
- https://cdimage.ubuntu.com/ubuntu-base/releases/
- https://doc.embedfire.com/linux/rk356x/build_and_deploy/zh/latest/building_image/ubuntu_rootfs/ubuntu_rootfs.html
- https://github.com/armbian/cache/releases/tag/0122

</details>

### konosubakonoakua · 2025-03-31T06:44:33Z

## Scripts
https://gist.github.com/konosubakonoakua/855ce64c961be1b31252fbe778116090

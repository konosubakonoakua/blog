---
number: 32
title: "[sw][linux][raspi] cross-compile linux kernel for raspberry pi"
state: open
url: https://github.com/konosubakonoakua/blog/issues/32
author: konosubakonoakua
createdAt: 2024-04-26T09:24:52Z
updatedAt: 2024-04-26T09:25:41Z
labels: []
---

# 32 [sw][linux][raspi] cross-compile linux kernel for raspberry pi

https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compiling-the-kernel

```bash
sudo apt install git bc bison flex libssl-dev make libc6-dev libncurses5-dev
sudo apt install crossbuild-essential-armhf
sudo apt install crossbuild-essential-arm64
git clone --depth=1 https://github.com/raspberrypi/linux

# 32-bit For Raspberry Pi 2, 3, 3+ and Zero 2 W, and Raspberry Pi Compute Modules 3 and 3+:
KERNEL=kernel7 && make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs -j8

# 64-bit For Raspberry Pi 3, 3+, 4, 400 and Zero 2 W, and Raspberry Pi Compute Modules 3, 3+ and 4:
KERNEL=kernel8 && make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
# 64-bit For Raspberry Pi 5
KERNEL=kernel_2712 && make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2712_defconfig

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs -j8

# menuconfig
sudo apt install libncurses5-dev
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

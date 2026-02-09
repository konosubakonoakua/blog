---
number: 153
title: "[fpga][zynq] petalinux devmem"
state: open
url: https://github.com/konosubakonoakua/blog/issues/153
author: konosubakonoakua
createdAt: 2025-02-26T15:55:30Z
updatedAt: 2025-03-25T09:23:37Z
labels: []
---

# 153 [fpga][zynq] petalinux devmem

- [how-to-enable-devmem-utility-in-petalinux](https://adaptivesupport.amd.com/s/question/0D52E00007G0uX5SAJ/how-to-enable-devmem-utility-in-petalinux?language=en_US)
- [very-low-level-access-with-peek-and-poke.html](https://mtvl-for-fpga.blogspot.com/2019/04/very-low-level-access-with-peek-and-poke.html)
- [devmem: another-very-low-access-method-with.html](https://mtvl-for-fpga.blogspot.com/2019/05/another-very-low-access-method-with.html)
- [從一開始的 Xilinx SoC 開發，PetaLinux 使用（三）](https://ys-hayashi.me/2021/09/xilinx-petalinux-03/)
- [how-to-design-and-access-a-memory-mapped-device-part-one](https://fpgacpu.wordpress.com/2013/05/28/how-to-design-and-access-a-memory-mapped-device-part-one/)

---

## Comments

### konosubakonoakua · 2025-02-27T07:32:04Z

- list all PL side devices
```bash
ls /sys/firmware/devicetree/base/amba_pl
```

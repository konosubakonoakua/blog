---
number: 105
title: "[hw][fpga] digilent jtag driver setup on windows"
state: open
url: https://github.com/konosubakonoakua/blog/issues/105
author: konosubakonoakua
createdAt: 2024-09-30T19:53:00Z
updatedAt: 2024-09-30T20:00:50Z
labels: []
---

# 105 [hw][fpga] digilent jtag driver setup on windows

## USB no response after cable plug in
- change to another usb port (back port on mother board is the best if PC)
## normal driver status
![image](../assets/105/1.png)
> [!CAUTION]
> The Description is `USB Serial Converter` & name is `Digilent USB Device`
> It's different name from linux version which is `Digilent Jtag ...`
## Reinstall digilent jtag driver
```bash
C:\Xilinx\Vivado\2024.1\data\xicom\cable_drivers\nt64\digilent\install_digilent.exe
```

---

## Comments

### konosubakonoakua · 2024-09-30T20:00:49Z

## Virtual Box
![image](../assets/105/2.png)
If you have driver that not working on windows, neither will work on vbox.
You will encounter attachment errors when trying to attach usb to vbox.

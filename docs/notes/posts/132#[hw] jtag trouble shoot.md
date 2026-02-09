---
number: 132
title: "[hw] jtag trouble shoot"
state: open
url: https://github.com/konosubakonoakua/blog/issues/132
author: konosubakonoakua
createdAt: 2024-12-31T02:53:44Z
updatedAt: 2025-03-17T10:38:07Z
labels: []
---

# 132 [hw] jtag trouble shoot

- DONE pin cannot pull up
  `ERROR: [Labtools 27-3165] End of startup status: LOW`
  - connect VREF to 3v3

---

## Comments

### konosubakonoakua · 2025-01-09T17:08:45Z

## vivado 2024.2 `program_ftdi` not found
do not use the desktop shortcut to start viavado, use terminal.
```bash
WARNING: [Common 17-259] Unknown Tcl command 'program_ftdi' sending command to the OS shell for execution. It is recommended to use 'exec' to send the command to the OS shell.
invalid command name "program_ftdi"
```
```bash
source /tools/Xilinx/Vivado/2024.2/settings64.sh && vivado
```
Then refer to https://adaptivesupport.amd.com/s/article/000037237?language=en_US
Download [ftdieeprom.tcl.gz](https://github.com/user-attachments/files/18365028/ftdieeprom.tcl.gz)
Then replace it in vivado scripts folder
- [program_ftdi](https://docs.amd.com/r/2022.1-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87/ug908-vivado-programming-debugging/%E7%94%A8%E4%BA%8E-Vivado-%E7%A1%AC%E4%BB%B6%E7%AE%A1%E7%90%86%E5%99%A8%E6%94%AF%E6%8C%81%E7%9A%84-FTDI-%E5%99%A8%E4%BB%B6%E7%BC%96%E7%A8%8B)
```bash
program_ftdi -write -f FT2232H -s 0000001 -v "@RF" -b "custom" -d "FT2232H JTAG Cable :D"
```
In case vivado cannot detect the device:
```bash
sudo rmmod ftdi_sio && sudo rmmod usbserial
```

### konosubakonoakua · 2025-01-09T17:25:38Z


## JTAG cable programmed by `program_ftdi` cannot be detected by vivado 2019.1
Maybe because udev rule files not okay.
```bash
sudo cp /tools/Xilinx/Vivado/2019.1/data/xicom/cable_drivers/lin64/install_script/install_drivers/52-xilinx-ftdi-usb.rules /etc/udev/rules.d/
```
Or
```bash
sudo /tools/Xilinx/Vivado/2019.1/data/xicom/cable_drivers/lin64/install_script/install_drivers/install_drivers

```

### konosubakonoakua · 2025-01-12T12:10:16Z

## references
- https://etherealwake.com/2024/06/xilinx-ftdi-jtag/

### konosubakonoakua · 2025-02-28T10:18:13Z

## vivado jtag flash download failed: `ftdi_read_data failed: usb bulk read failed`
- [error-in-launching-program-from-sdk-20182](https://adaptivesupport.amd.com/s/question/0D52E00006hpi4eSAA/error-in-launching-program-from-sdk-20182?language=en_US)
  - try another ftdi jtag debugger
  - confirm powersupply
  - installed wrong ftdi driver
  - try to inspect component, e.g. 12MHz crystal (MUST satisfy +-30ppm)


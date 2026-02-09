---
number: 74
title: "[hw][fpga] Xilinx ISE 14.7 setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/74
author: konosubakonoakua
createdAt: 2024-07-09T06:37:54Z
updatedAt: 2024-10-05T11:15:46Z
labels: ["FPGA"]
---

# 74 [hw][fpga] Xilinx ISE 14.7 setup

## deps
```bash
sudo apt-get install libncurses5 
```
## download
https://www.xilinx.com/downloadNav/vivado-design-tools/archive-ise.html
## install
```bash
tar -xvf Xilinx_ISE_DS_Lin_14.7_1015_1.tar
sudo ./xsetup
```
![image](../assets/74/1.png)


---

## Comments

### konosubakonoakua · 2024-09-23T03:51:46Z

## create desktop file
impact.desktop
```ini
[Desktop Entry]
Version=1.0
Name=Xilinx ISE Impact
Comment=Xilinx ISE 14.7 Impact
Exec=bash -c "source /tools/Xilinx/14.7/ISE_DS/settings64.sh && impact"
Icon=/tools/Xilinx/14.7/ISE_DS/ISE/data/images/impact.png
StartupNotify=true
Type=Application
Categories=Application;
```
ise.desktop
```ini
[Desktop Entry]
Version=1.0
Name=Xilinx ISE
Comment=Xilinx ISE 14.7
Exec=bash -c "source /tools/Xilinx/14.7/ISE_DS/settings64.sh && ise"
Icon=/tools/Xilinx/14.7/ISE_DS/ISE/data/images/pn-ise.png
StartupNotify=true
Type=Application
Categories=Application;
```

Then, copy them to desktop or `~/.local/share/applications`.
```bash
cp -f ise.desktop impcat.desktop ~/.local/share/appliations
```

Or just use commands below:
```bash
mkdir -p ~/.local/share/applications
cd ~/.local/share/applications/

cat <<EOF > impact.desktop
[Desktop Entry]
Version=1.0
Name=Xilinx ISE Impact
Comment=Xilinx ISE 14.7 Impact
Exec=bash -c "source /tools/Xilinx/14.7/ISE_DS/settings64.sh && impact"
Icon=/tools/Xilinx/14.7/ISE_DS/ISE/data/images/impact.png
StartupNotify=true
Type=Application
Categories=Application;
EOF

cat <<EOF > ise.desktop
[Desktop Entry]
Version=1.0
Name=Xilinx ISE
Comment=Xilinx ISE 14.7
Exec=bash -c "source /tools/Xilinx/14.7/ISE_DS/settings64.sh && ise"
Icon=/tools/Xilinx/14.7/ISE_DS/ISE/data/images/pn-ise.png
StartupNotify=true
Type=Application
Categories=Application;
EOF

```

### konosubakonoakua · 2024-09-23T03:52:16Z

## install Digilent JtagSMT2 driver

first,
```bash
sudo mkdir -p /etc/hotplug/usb
cd /tools/Xilinx/14.7/ISE_DS/common/bin/lin64/digilent
sudo ./install_digilent.sh
# selected libcsddigilent install to /home/rf/.cse
# or try /tools/Xilinx/14.7/ISE_DS/ISE (do not)
```

if don't create `/etc/hotplug/usb` first, then
```
Successfully installed Adept Runtime configuration "/etc/digilent-adept.conf".
Installing hotplug script.....
cp: cannot create regular file '/etc/hotplug/usb/digilentusb': No such file or directory
error: failed to install hotplug script "/etc/hotplug/usb/digilentusb"
error: failed to install Adept Runtime, exitcode = 4
See "/tools/Xilinx/14.7/ISE_DS/common/bin/lin64/digilent/digilent.adept.runtime_2.13.1-x86_64/README" for information regarding installation.
```

maybe, this one is also needed.
```bash
sudo cp -f /tools/Xilinx/14.7/ISE_DS/common/bin/lin64/digilent/digilent.adept.runtime_2.13.1-x86_64/52-digilent-usb.rules /etc/udev/rules.d/
sudo chmod 644 /etc/udev/rules.d/52-digilent-usb.rules
```

finally,
```
Installing component: libCseDigilent
Installer for Plugins for Xilinx Tools
In which directory should plugins be installed? [/root/.cse] /tools/Xilinx/14.7/ISE_DS/ISE
Plugins will be installed in the following directory: "/tools/Xilinx/14.7/ISE_DS/ISE"
    Note: The "XIL_CSE_PLUGIN_DIR" environment variable must be set to "/tools/Xilinx/14.7/ISE_DS/ISE" prior to using the Xilinx Tools.
Found plugin "libCseDigilent" version "14.1" by "Digilent".
    Successfully installed plugin in "/tools/Xilinx/14.7/ISE_DS/ISE/lin64/14.1/plugins/Digilent/libCseDigilent".
 
```

in impcat
```
INFO:iMPACT - Digilent Plugin: Plugin Version: 2.4.4
INFO:iMPACT - Digilent Plugin: found 1 device(s).
INFO:iMPACT - Digilent Plugin: opening device: "JtagSmt2", SN:210251190311
INFO:iMPACT - Digilent Plugin: User Name: JtagSmt2
INFO:iMPACT - Digilent Plugin: Product Name: Digilent JTAG-SMT2
INFO:iMPACT - Digilent Plugin: Serial Number: 210251190311
INFO:iMPACT - Digilent Plugin: Product ID: 31000154
INFO:iMPACT - Digilent Plugin: Firmware Version: 0108
INFO:iMPACT - Digilent Plugin: JTAG Port Number: 0
INFO:iMPACT - Digilent Plugin: JTAG Clock Frequency: 10000000 Hz
Attempting to identify devices in the boundary-scan chain configuration...
INFO:iMPACT - Current time: 24-9-23 PM2:02
```

### konosubakonoakua · 2024-09-23T06:42:15Z

trouble shoot
if busy, then `sudo rm -f /tmp/digilent-adept2-*`

### digilent utilities (do not install, will break the impact)
- https://digilent.com/reference/software/adept/start
  - install adept [runtime](https://lp.digilent.com/complete-adept-runtime-download) & [utility](https://lp.digilent.com/complete-adept-utilities-download)
```bash
rf@<hidden>:~$ sudo djtgcfg enum
Found 1 device(s)

Device: JtagSmt2
    Device Transport Type: 00020001 (USB)
    Product Name:          Digilent JTAG-SMT2
    User Name:             JtagSmt2
    Serial Number:         210251190311

rf@<hidden>:~$ sudo djtgcfg init -d JtagSmt2
Initializing scan chain...
Found Device ID: 13631093

Found 1 device(s):
    Device 0: XC7A100T
```
uninstall it.
```bash
sudo apt remove digilent.adept.runtime
```

### konosubakonoakua · 2024-09-23T07:00:12Z

## in case you want to use DLC9G usb-driver
https://github.com/konosubakonoakua/blog/issues/76

## docs
- https://docs.amd.com/v/u/en-US/ug1655-ise-documentation

### konosubakonoakua · 2024-10-05T11:15:45Z

## Tutorial
- [ISE14.7手把手使用教程：建立工程、仿真、下载bit流、程序固化、以及一些常见的坑](https://blog.csdn.net/yang_jiangning/article/details/123546078)

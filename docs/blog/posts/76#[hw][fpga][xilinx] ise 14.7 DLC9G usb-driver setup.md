---
number: 76
title: "[hw][fpga][xilinx] ise 14.7 DLC9G usb-driver setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/76
author: konosubakonoakua
createdAt: 2024-08-03T10:57:08Z
updatedAt: 2024-09-23T09:13:48Z
labels: ["FPGA", "ZYNQ", "Hardware"]
---

# 76 [hw][fpga][xilinx] ise 14.7 DLC9G usb-driver setup

## troubleshooting
### Xilinx JTAG Programmer (USB) cable not found
> [!IMPORTANT]
> ***Only Platform Cable USB II
 Platform Cable USB Model: DLC9G is supported!!!!!!!!!!!!***
> ~~do not use digilent ft232 jtag cables~~

First of all, this guide assumes you have installed Xilinx ISE (version 14.7 is used here) into the default path of /opt/Xilinx/ISE/

Next, you will need to have GIT installed to get the required libraries. This approach does not use the official Xilinx libraries but a replica of them. You will also need libusb-dev which is required in the compiling of the drivers. You will also need build-essential metapackage for the compiler. On a 64-bit host, you will need to get the 32-bit version of libc6-dev-i386.

**On 32-bit**
```bash
sudo apt-get install gitk git-gui libusb-dev build-essential libc6-dev fxload
```
**On 64-bit**
```bash
sudo apt-get install gitk git-gui libusb-dev build-essential libc6-dev-i386 fxload
```
> Philipp HÃrauf reports there is a need to install ia32-libs-dev on some 64-bit systems, though I didn’t require this.

**Clone usb-driver:**
```bash
sudo git clone git://git.zerfleddert.de/usb-driver /opt/Xilinx/ISE/usb-driver
```

**Compiling the Driver**
This step is once simple command, but the most important. It compiles the cable driver required. There are two versions depending on the version (32-/64-bit) of Xilinx ISE you have. Firstly change into the source directory created in the previous step. Then you need to issue the make command.

```bash
cd usb-driver/
sudo make
sudo cp libusb-driver.so libusb-driver-DEBUG.so /usr/local/lib
### To solve windrvr6 not found
sudo sh -c 'echo "export LD_PRELOAD=/usr/local/lib/libusb-driver.so" >> /etc/bash.bashrc'

chmod +x ./setup_pcusb
./setup_pcusb /opt/Xilinx/ISE/14.7/ISE_DS/ISE/

sudo cp /opt/Xilinx/ISE/14.7/ISE_DS/ISE/bin/lin64/xusbdfwu.rules /etc/udev/rules.d/50-xusbdfwu.rules
sudo sed -i -e 's/TEMPNODE/tempnode/' -e 's/SYSFS/ATTRS/g' -e 's/BUS/SUBSYSTEMS/' /etc/udev/rules.d/50-xusbdfwu.rules
sed /opt/Xilinx/ISE/14.7/ISE_DS/ISE/bin/lin64/xusbdfwu.rules -e 's:TEMPNODE:tempnode:g' | sed 's:BUS=="usb", ::g' | sed 's:SYSFS:ATTR:g' | sudo tee /etc/udev/rules.d/50-xusbdfwu.rules 1>/dev/null

sudo cp -f /opt/Xilinx/ISE/14.7/ISE_DS/ISE/bin/lin64/xusb*.hex /usr/share/
sudo chmod 644 /usr/share/xusb*.hex
sudo sed -i -e 's/#!\/bin\/sh/#!\/bin\/bash/' /opt/Xilinx/ISE/14.7/ISE_DS/PlanAhead/bin/planAhead
sudo sed -i -e 's/#!\/bin\/sh/#!\/bin\/bash/' /opt/Xilinx/ISE/14.7/ISE_DS/PlanAhead/bin/loader
sudo sed -i -e 's/#!\/bin\/sh/#!\/bin\/bash/' /opt/Xilinx/ISE/14.7/ISE_DS/ISE/bin/lin64/unwrapped/analyzer
sudo /bin/udevadm control --reload-rules
```
then we can use `xmd` to verify the connections
```bash
source /opt/Xilinx/ISE/14.7/ISE_DS/settings64.sh
xmd

XMD% connect mdm

JTAG chain configuration
--------------------------------------------------
Device   ID Code        IR Length    Part Name
 1       13631093           6        XC7A100T
ERROR: Could not detect MDM peripheral on hardware. Please check:
	1. If FPGA is configured correctly
	2. MDM Core is instantiated in the design
	3. If the correct FPGA is referred, in case of multiple FPGAs
	4. If the correct MDM is referred, in case of a multiple MDM system

```
with id code & part name, we can say that the connection between jtag cable & fpga chip is good.

Then we open iMPACT-->double click `Boundary Scan`-->Right click on blank area: `initialize chain`--->see the jtag daisy link.

**links**
- https://www.george-smart.co.uk/fpga/xilinx_jtag_linux/
- https://support.xilinx.com/s/article/64361?language=en_US


---

## Comments

### konosubakonoakua · 2024-08-05T08:10:06Z

### cannot find jtag device (in virtualbox)
- try to plug out then plug in jtag usb device first
- sometimes virtual box will not detect jtag connected.
- it's better enable option to automatically connect jtag to vbox
    ![image](../assets/76/1.png)
#### you should, 
- on host side:
  - Check linux usb device by `sudo lsusb`
  - Check and replace host USB cable.
  - Check and change the host USB port.
  - Try on another working board
  - Try another good tag device
  - Reboot sometimes helps too
- on target side:
  - Check the target fpga borad power status.
  - Power off target board before remove/attach jtag cable
  - ***Check the boot option (jtag mode) on the FPGA board.***



### konosubakonoakua · 2024-08-29T14:42:25Z

- https://chaosink.github.io/2016/05/20/2016-05-20-Module-windrvr6-is-not-loaded-Please-reinstall-the-cable-drivers-See-Answer-Record-22648/
- https://wiki.gentoo.org/wiki/Xilinx_USB_JTAG_Programmers#Digilent_Xilinx_USB_JTAG_cable
- https://rmdir.de/~michael/xilinx/
- https://digilent.com/reference/software/digilent-plugin-xilinx-tools/start?redirect=2
- https://lp.digilent.com/complete-adept-runtime-download
- https://digilent.com/reference/programmers/jtag-smt2-nc/reference-manual

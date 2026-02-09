---
number: 93
title: "[hw][fpga] linux Vivado 2019.1 setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/93
author: konosubakonoakua
createdAt: 2024-09-23T06:25:31Z
updatedAt: 2025-01-09T17:25:29Z
labels: ["FPGA", "ZYNQ", "Linux"]
---

# 93 [hw][fpga] linux Vivado 2019.1 setup

## download
https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive.html

## deps
```bash
sudo apt install libtinfo5 libncurses5
sudo apt-get install -y python gawk gcc git make net-tools libncurses5-dev tftpd zlib1g:i386 libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib build-essential screen pax gzip

```

## install
```bash
cd ~/Downloads
tar xvfz Xilinx_Vivado_SDK_2019.1_0524_1430.tar.gz
cd Xilinx_Vivado_SDK_2019.1_0524_1430
sudo ./xsetup
```

## create desktop files
on ubuntu 22, the desktop file maybe will not work immediately, if not, try reboot or logout.
```bash
mkdir -p ~/.local/share/applications
cd ~/.local/share/applications/

cat <<EOF > vivado2019.1.desktop
[Desktop Entry]
Type=Application
Name=Vivado 2019.1
Exec=bash -c "source /tools/Xilinx/Vivado/2019.1/settings64.sh && /tools/Xilinx/Vivado/2019.1/bin/vivado"
Icon=/tools/Xilinx/Vivado/2019.1/doc/images/vivado_logo.png
StartupNotify=true
Categories=Application;
EOF
```

---

## Comments

### konosubakonoakua · 2024-09-23T07:01:34Z

## jtag troubleshoot
can both use Platfrom cable usb (DLC9G) & Digilent JtagSMT2.
- But DLC9G may raise hw_server too old error if last time connection is failed or cancelled
- locked by another hw_server:
    ```bash
    ERROR: [Labtoolstcl 44-494] There is no active target available for server at localhost.
     Targets(s) ", jsn-JTAG-SMT2-210251190311" may be locked by another hw_server.
    ```
    
    ```bash
    sudo rm -rf /tmp/digilent*
    kill $(ps -A | grep hw_server | awk '{ print $1 }')
    ```
### if you are using virtual box
#### cannot find jtag device (in virtualbox)
- try to plug out then plug in jtag usb device first
- sometimes virtual box will not detect jtag connected.
- it's better enable option to automatically connect jtag to vbox
    ![image](../assets/93/1.png)
##### you should, 
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


#### vivado 2019 hw_server error (in virtualbox linux mint22)
```bash
connect_hw_server
INFO: [Labtools 27-2285] Connecting to hw_server url TCP:localhost:3121
INFO: [Labtools 27-2222] Launching hw_server...
INFO: [Labtools 27-2221] Launch Output:

****** Xilinx hw_server v2019.1
  **** Build date : May 24 2019 at 15:06:40
    ** Copyright 1986-2019 Xilinx, Inc. All Rights Reserved.


ERROR: [Labtoolstcl 44-494] There is no active target available for server at localhost.
 Targets(s) ", jsn-JTAG-SMT2-210251190311" may be locked by another hw_server.
disconnect_hw_server localhost:3121
```

**Solution:**
- set usb3.0 in virtualbox
    ![image](../assets/93/2.png)
- try close Server then autoconnect
    ![image](../assets/93/3.png)
- close hardware manager first
- try `sudo rm -rf /tmp/digilent* && ps -A | grep -q 'hw_server' && pkill -f hw_server`
- try reopen hardware manager
- try reboot

**Finally**:
![image](../assets/93/4.png)

### if you are using vmware
sorry, I haven't tried it yet.


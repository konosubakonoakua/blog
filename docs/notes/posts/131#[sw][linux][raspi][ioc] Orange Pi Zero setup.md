---
number: 131
title: "[sw][linux][raspi][ioc] Orange Pi Zero setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/131
author: konosubakonoakua
createdAt: 2024-12-24T09:00:39Z
updatedAt: 2025-02-10T03:25:59Z
labels: []
---

# 131 [sw][linux][raspi][ioc] Orange Pi Zero setup

### image
<details>

- [armbian](https://www.armbian.com/orange-pi-zero/)

```powershell
scoop install rufus
```

</details>

### apt mirrors

<details>

```bash
sudo cp -f /etc/apt/sources.list /etc/apt/sources.list.bak
sudo tee /etc/apt/sources.list << 'EOF'
deb https://mirrors.bfsu.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb https://mirrors.bfsu.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb https://mirrors.bfsu.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

# deb-src https://mirrors.bfsu.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.bfsu.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.bfsu.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
EOF

sed -i.bak 's#http://beta.armbian.com#https://mirrors.bfsu.edu.cn/armbian#g' /etc/apt/sources.list.d/armbian.list
#sed -i.bak 's#http://apt.armbian.com#https://mirrors.bfsu.edu.cn/armbian#g' /etc/apt/sources.list.d/armbian.list

sudo apt update && sudo apt upgrade
```

</details>

### samba & tftp

<details>

[raspi docs samba](https://www.raspberrypi.com/documentation/computers/remote-access.html#samba-smbcifs)
```bash
sudo apt update
sudo apt install samba samba-common-bin smbclient cifs-utils
```
```bash
cd ~
mkdir shared
chmod 0740 shared
sudo smbpasswd -a opi
```
```bash
sudo tee -a /etc/samba/smb.conf << 'EOF'
[shared]
    path = /home/rpi/shared
    writable = yes
    browsable = yes
    public = yes
    create mask = 0777
    max connections = 10
EOF
sudo sed -i '/map to guest = bad user/s/^/# /' /etc/samba/smb.conf
```
```bash
sudo systemctl restart smbd
```
```bash
# trouble shoot
rm -rf /var/log/samba/log.smbd
sudo rm -rf /var/lib/samba/registry.tdb
```

[Test setup (TFTP)](https://www.raspberrypi.com/documentation/computers/remote-access.html#test-setup)

</details>

### [Networking](https://docs.armbian.com/User-Guide_Networking/#networking)

<details>

```bash
# only for minimal images
sudo tee /etc/netplan/20-static-ip.yaml << EOF
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
      - 192.168.137.201/24
      routes:
      - to: default
        via: 192.168.137.1
      nameservers:
       addresses:
         - 114.114.114.114
         - 8.8.8.8
EOF
```
```bash
sudo chmod 600 /etc/netplan/*.yaml
sudo netplan try
sudo netplan apply
```

</details>

### Install fzf/zellij/zoxide

<details>

Download them from windows side
- [fzf](https://github.com/junegunn/fzf/releases)

Or just use samba on windows side.

</details>


### python

<details>

- install pkgs
  ```bash
  sudo apt install python3-dev python3-pip python3-venv
  ```
- add pypi mirrors
  ```bash
  # pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple
  pip config set global.index-url https://www.piwheels.org/simple
  ```
- create venvs
  ```bash
  mkdir -p ~/.virtualenvs
  cd ~/.virtualenvs
  python3 -m venv epics
  source ~/.virtualenvs/epics/bin/activate
  ```
- failed to build wheels, then use piwheels
  ```bash
  pip3 install numpy -i https://www.piwheels.org/simple
  ```
- numpy failed to import, cannot find openblas
  ```bash
  sudo apt install libopenblas-dev
  ```

</details>




---

## Comments

### konosubakonoakua · 2024-12-24T11:54:54Z

### epics on orange pi zero
<details>

```bash
sudo apt -y install git net-tools build-essential re2c libnet-dev libpcap-dev libusb-1.0-0-dev
```
- https://epics.anl.gov/download/base/index.php

```bash
mkdir $HOME/EPICS
cd $HOME/EPICS
wget https://epics.anl.gov/download/base/base-7.0.8.1.tar.gz
tar xf base-7.0.8.1.tar.gz && ln -s base-7.0.8.1 epics-base
cd epics-base
make -j2
~/EPICS/bin/linux-arm/softIoc
```
> [!IMPORTANT]
> You need to set `PYEPICS_LIBCA` to load the correct `libca.so` on rpi
> Refer to: [pyepics issue 238](https://github.com/pyepics/pyepics/issues/238)
```bash
cat >> ~/.bashrc << 'EOF'

export EPICS_BASE=$HOME/EPICS/epics-base
export EPICS_HOST_ARCH=linu-arm
export PYEPICS_LIBCA=$HOME/EPICS/epics-base/lib/linux-arm/libca.so

EOF
```
#### pythonsoftioc on raspi
```bash
# because it will install 7.0.7.99.0.2 which will not work with latest softioc
uv pip install epicscorelibs==7.0.7.99.1.1
uv pip install softioc
```
Then try with [this test](https://diamondlightsource.github.io/pythonSoftIOC/master/tutorials/creating-an-ioc.html).

#### pcaspy on raspi
```bash
sudo apt install -y swig
```

```bash
mkdir ~/EPICS/support
cd ~/EPICS/support
wget https://github.com/epics-modules/pcas/archive/refs/tags/v4.13.3.tar.gz
mv v4.13.3.tar.gz pcas-4.13.13.tar.gz
tar xvzf pcas-4.13.3.tar.gz && rm pcas-4.13.3.tar.gz
cd ./pcas-4.13.3
```

```bash
cat > configure/RELEASE.local << EOF
EPICS_BASE = ${EPICS_BASE}
EOF
```

```bash
make -j4
```

```bash
cat >> ~/.bashrc << EOF
export PCAS=~/EPICS/support/pcas-4.13.3
EOF
```

```bash
source ~/.bashrc
```

```bash
uv pip install pcaspy
```
If encountered with: `ImportError: libca.so.4.14.3: cannot open shared object file: No such file or directory`, Check version first:

```bash
caget -V
```
```bash
ldd ~/.virtualenvs/opi/lib/python3.11/site-packages/pcaspy/_cas.cpython-311-armhf-linux-gnu.so
```
If epics version not matched with pcas build, then use `ln -s` to make softlinks for shared libs.

```bash
cd $EPICS_BASE/lib/linux-arm/
ln -s libca.so.4.14.4 libca.so.4.14.3
ln -s libCom.so.3.23.1 libCom.so.3.23.0
```

</details>


### konosubakonoakua · 2024-12-24T14:21:46Z

### swapfile

<details>

```bash
## add
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
```bash
## remove
sudo swapoff /swapfile
sudo rm /swapfile
```

</details>


### konosubakonoakua · 2024-12-24T15:06:02Z

### autostart
#### rc-local

<details>

```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>IP address is %s\n" "$_IP"
fi

### add your commands here ###

# keep this line
exit 0
```

</details>

#### systemd

<details>

- Example 
  ```bash
  sudo tee /etc/systemd/system/ioc-instr-cs-keithley-6221.service << EOF
  [Unit]
  Description=softioc for keithley current source 6221
  After=network.target
  StartLimitIntervalSec=60
  StartLimitBurst=5
  
  [Service]
  ExecStart=/home/opi/shared/iocs/ioc-instr-cs-keithley-6221/pve/softioc_keithley6221.sh
  WorkingDirectory=/home/opi/shared/iocs/ioc-instr-cs-keithley-6221/pve/
  StandardOutput=inherit
  StandardError=inherit
  Restart=always
  User=opi
  
  [Install]
  WantedBy=multi-user.target
  EOF
  
  ```
- Enable & Start
  ```bash
  sudo systemctl enable ioc-instr-cs-keithley-6221.service
  sudo systemctl start ioc-instr-cs-keithley-6221.service
  ```
- Logging
  ```bash
  journalctl -u ioc-instr-cs-keithley-6221.service -f
  ```
- Restart
  ```bash
  sudo systemctl reset-failed ioc-instr-cs-keithley-6221.service
  sudo systemctl restart ioc-instr-cs-keithley-6221.service
  ```
- Reload
  ```bash
  sudo systemctl daemon-reload
  ```

</details>

---
number: 165
title: "[fpga][xilinx] vivado/petalinux 2021.x setup notes"
state: open
url: https://github.com/konosubakonoakua/blog/issues/165
author: konosubakonoakua
createdAt: 2025-03-24T14:17:21Z
updatedAt: 2025-04-14T01:41:40Z
labels: []
---

# 165 [fpga][xilinx] vivado/petalinux 2021.x setup notes

## prepare

<details>

[OS Requirement](https://docs.amd.com/r/2021.1-English/ug973-vivado-release-notes-install-license/Supported-Operating-Systems)
- Microsoft Windows Professional/Enterprise 10.0 1809 Update; 10.0 1903 Update; 10.0 1909 Update; 10.0 2004 Update
- Red Hat Enterprise Workstation/Server 7.4, 7.5, 7.6, 7.7, 7.8, 7,9, 8.1, 8.2, and 8.3 (64-bit), English/Japanese
- CentOS 7.4, 7.5, 7.6, 7.7, 7.8, 8.1, 8.2, and 8.3 (64-bit), English/Japanese
- SUSE Linux Enterprise 12.4 and 15.2 (64-bit), English/Japanese
- Amazon Linux 2 AL2 LTS (64-bit)
- Ubuntu Linux 16.04.5 LTS;16.04.6 LTS; 18.04.1 LTS; 18.04.2 LTS, 18.04.3 LTS; 18.04.4 LTS; and 20.04 LTS, 20.04.1 LTS (64-bit), English/Japanese

```bash
sudo adduser $USER dialout
sudo dpkg --add-architecture i386
sudo dpkg-reconfigure dash # use tab to select no
```
```bash
sudo apt-get update
```
```bash
sudo apt install \
	lib32stdc++6 \
	libgtk2.0-0:i386 \
	libfontconfig1:i386 \
	libx11-6:i386 \
	libxext6:i386 \
	libxrender1:i386 \
	libsm6:i386 \
	libqtgui4:i386 \
	xinetd \
	iproute2 \
	gawk \
	gcc \
	net-tools \
	ncurses-dev \
	openssl \
	libssl-dev \
	flex \
	bison \
	xterm \
	autoconf \
	libtool \
	texinfo \
	zlib1g-dev \
	gcc-multilib \
	build-essential \
	automake \
	screen \
	putty \
	pax \
	g++ \
	python3-pip \
	xz-utils \
	python3-git \
	python3-jinja2 \
	python3-pexpect \
	debianutils \
	iputils-ping \
	libegl1-mesa \
	libsdl1.2-dev \
	pylint3 \
	python3 \
	cpio \
	tftpd-hpa \
	gnupg \
	zlib1g:i386 \
	haveged \
	perl \
	liberror-perl \
	mtd-utils \
	xtrans-dev \
	libxcb-randr0-dev \
	libxcb-xtest0-dev \
	libxcb-xinerama0-dev \
	libxcb-shape0-dev \
	libxcb-xkb-dev \
	openssh-server \
	util-linux \
	sysvinit-utils \
	cython \
	google-perftools
```
taken from pynq repo:
```bash
## petalinux & vivado deps
# Install a bunch of packages we need

if [ $(lsb_release -rs) == "18.04" ]; then
    rel_deps="libncurses5-dev lib32ncurses5"
else
    rel_deps="libncurses6 lib32ncurses6"
fi

read -d '' PACKAGES <<EOT
bc
libtool-bin
gperf
bison
flex
texi2html
texinfo
help2man
gawk
libtool
build-essential
automake
libglib2.0-dev
libfdt-dev
device-tree-compiler
qemu-user-static
binfmt-support
multistrap
git
lib32z1
lib32stdc++6
libssl-dev
kpartx
dosfstools
nfs-common
zerofree
u-boot-tools
rpm2cpio
curl
libsdl1.2-dev
libpixman-1-dev
libc6-dev
chrpath
socat
zlib1g-dev
zlib1g:i386
unzip
rsync
python3-pip
gcc-multilib
xterm
net-tools
libidn11
ninja-build
python3-testresources
${rel_deps}
EOT
set -e

sudo apt-get update

sudo add-apt-repository -y ppa:ubuntu-toolchain-r/ppa
sudo apt-get update
sudo apt-get install -y $PACKAGES
```
```
```bash
sudo mkdir /tftpboot
sudo chmod -R 777 /tftpboot
sudo chown -R nobody /tftpboot

sudo tee /etc/xinetd.d/tftp1 << EOF
service tftp 
    {
    protocol = udp 
    port = 69 
    socket_type = dgram 
    wait = yes 
    user = nobody 
    server = /usr/sbin/in.tftpd 
    server_args = /tftpboot 
    disable = no
    }
EOF

sudo /etc/init.d/xinetd stop
sudo /etc/init.d/xinetd start
```

</details>

---

## Comments

### konosubakonoakua · 2025-03-25T05:36:13Z

## install 

<details>

### vivado 2021
```bash
sudo mkdir /tools/
sudo chmod 755 /tools/
sudo chown rf:rf /tools/
mkdir /tools/Xilinx
/mnt/winshared/linux/Xilinx_Unified_2021.1_0610_2318/xsetup
```

After vivado installtion, in tcl console:
```tcl
config_webtalk -user off
config_webtalk -info
```
In bash console:
```bash
cd /tools/Xilinx/Vivado/2021.1/data/xicom/cable_drivers/lin64/install_script/install_drivers
sudo ./install_drivers
```
In License manager:
`Help`/`Manage License`/`Get License`/`Load License`/`Copy License`/`Select License File`/`Open Your Lic File`

[2018.1_license.zip](https://github.com/user-attachments/files/19499809/vivado_2018.1_license.zip)

### petalinux 2021
```bash
sudo mkdir -p /tools/Xilinx/PetaLinux/2021.1/
sudo chmod -R 755 /tools/Xilinx/PetaLinux/2021.1/
sudo chown -R rf:rf /tools/Xilinx/PetaLinux/2021.1/ # user:user
sudo vmhgfs-fuse .host:/ /mnt/winshared -o allow_other
/mnt/winshared/linux/petalinux-v2021.1-final-installer.run -d /tools/Xilinx/PetaLinux/2021.1/
```

</details>

### konosubakonoakua · 2025-03-27T07:25:11Z

## sstate

<details>

### cifs
```bash
mount | grep /mnt
sudo umount /mnt
sudo mkdir -p /mnt/petalinux/sstate
sudo apt install -y cifs-utils
```
```bash
sudo mount -t cifs //rf-pc-shiro.local/D/tools/petalinux/sstate /mnt/petalinux/sstate -o username=RF-PC-SHIRO
```
### petalinux/project-spec/configs/config
```makefile
#
# Add pre-mirror url 
#
CONFIG_PRE_MIRROR_URL="file:///mnt/petalinux/sstate/2021.1/downloads/"

#
# Local sstate feeds settings
#
CONFIG_YOCTO_LOCAL_SSTATE_FEEDS_URL="/mnt/petalinux/sstate/2021.1/aarch64"
# CONFIG_YOCTO_NETWORK_SSTATE_FEEDS is not set
CONFIG_YOCTO_BB_NO_NETWORK=y
# CONFIG_YOCTO_BUILDTOOLS_EXTENDED is not set

```

### meta-user/conf/petalinuxbsp.conf
```makefile
DL_DIR = "/impcas/cache/petalinux/sstate/2021.1/downloads"
SSTATE_DIR = "/impcas/cache/petalinux/sstate/2021.1//aarch64"
```

petalinux sstate issues: https://github.com/konosubakonoakua/blog/issues/152#issuecomment-2800268052

</details>

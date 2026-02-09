---
number: 97
title: "[hw][fpga][xilinx] petalinux 2019.1 setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/97
author: konosubakonoakua
createdAt: 2024-09-23T14:46:59Z
updatedAt: 2024-10-09T03:51:25Z
labels: ["FPGA", "ZYNQ", "Linux"]
---

# 97 [hw][fpga][xilinx] petalinux 2019.1 setup

## Dependencies
Switch to bash
```bash
sudo dpkg-reconfigure dash #####choose NO
```
Install dependecies
```bash
sudo apt update
sudo apt -y install libtool-bin \
  python2 gcc make perl tofrodos iproute2 gawk git \
  xvfb net-tools tftp-hpa tftpd-hpa flex bison \
  screen pax gzip tar gnupg wget diffstat chrpath \
  socat xterm autoconf libtool tar unzip texinfo \
  gcc-multilib build-essential libselinux1 \
  zlib1g-dev zlib1g-dev:i386 \
  libsdl1.2-dev libglib2.0-dev \
  libssl-dev libncurses5-dev libncursesw5 \
  nfs-kernel-server openssh-server \
  expect
```
Config tftp server (optional).
```bash
sudo mkdir -p /tftpboot
sudo chown rf:rf /tftpboot
chmod 777 /tftpboot
```
```bash
sudo cp -f /etc/default/tftpd-hpa /etc/default/tftpd-hpa.bak

sudo tee /etc/default/tftpd-hpa << EOF
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="-l -c -s"
EOF

sudo service tftpd-hpa restart
```


Install ssh server
```bash
sudo apt-get install openssh-server
# vim /etc/ssh/sshd_config
```

---

## Comments

### konosubakonoakua · 2024-09-23T17:50:48Z

## petalinux installation (ubuntu 18 & below)
### create a fake python2 pkg status
```bash
sudo tee -a /var/lib/dpkg/status << EOF

Package: python
Status: install ok installed
Maintainer: Fake Entry <fake@example.com>
Architecture: all
Version: 2.7.17
Description: fake package for petalinux

EOF

sudo ln -s /usr/bin/python2.7 /usr/bin/python
```
If not, petalinux 2019 won't find python, and will raise error.
```
ERROR: You are missing the following system tools required by PetaLinux:
 - python
```
### run the installer
```bash
export INSTALLER=~/Downloads/petalinux-v2019.1-final-installer.run
$INSTALLER /tools/petalinux/2019.1/
# need confirm with licenses
```


### konosubakonoakua · 2024-09-24T13:20:45Z

## sstate cache
create /tools/petalinux if not exists
```bash
sudo mkdir -p /tools/petalinux
sudo chown rf:rf /tools/petalinux
```
```bash
mkdir -p /tools/petalinux/sstate/2019.1
tar xvzf ~/Downloads/sstate-rel-v2019.1.tar.gz -C /tools/petalinux/sstate/2019.1
```

### konosubakonoakua · 2024-09-24T14:37:10Z

## trouble shoot
### build failed (libc version)
I'm using ubuntu22, so I use docker instaled. (follow #84)
```bash
git clone https://github.com/konosubakonoakua/petalinux-docker-2019.1
```

## petalinux on docker (ubuntu 22)
build
```bash
docker build \
  --build-arg UBUNTU_MIRROR=mirrors.bfsu.edu.cn \
  --build-arg PETA_VERSION=2019.1 \
  --build-arg PETA_RUN_FILE=petalinux-v2019.1-final-installer.run \
  -t petalinux:2019.1 \
  .
```
launch
```bash
docker run -ti --rm \
  -e DISPLAY=$DISPLAY --net="host" \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v $HOME/.Xauthority:/home/vivado/.Xauthority \
  -v /tools/petalinux/sstate/2019.1:/tools/petalinux/sstate/2019.1 \
  -v /tftpboot:/tftpboot \
  -v $(pwd):/home/vivado/project \
  petalinux:2019.1 \
  /bin/bash
```
oneliner
```bash
tee -a ~/.bashrc << EOF
alias petadocker2019v1='docker run -it --rm -e DISPLAY=$DISPLAY --net="host" -v /tmp/.X11-unix:/tmp/.X11-unix -v $HOME/.Xauthority:/home/vivado/.Xauthority -v /tools/petalinux/sstate/2019.1:/tools/petalinux/sstate/2019.1 -v /tftpboot:/tftpboot -v $(pwd):/home/vivado/project petalinux:2019.1 /bin/bash'
EOF
source ~/.bashrc
```

If you are runing docker in virtual machine shared folders, then the `-v` mount will fail.
If you do `ls` in docker, You will get:
```bash
vivado@ubuntu22-vb:~/project$ ls
ls: cannot open directory '.': Permission denied
```
Solution:
- do not run docker in these folders.
  - https://forums.docker.com/t/need-docker-containers-to-rwx-onto-virtualbox-shared-folders/73454
- use smb
  - mount smb first, follow #82
  - mount mounted smb folder by `-v` to docker
  - You can use this way to host sstate-cache on only one machine to save you drive.

Save docker images:
- https://docs.docker.com/reference/cli/docker/image/save/
    
    ```bash
    docker save petalinux:2019.1 | gzip > petalinux-2019.1-docker.tar.gz
    ```
    or
    ```bash
    docker save petalinux:2019.1 > petalinux-2019.1-docker.tar
    pigz -9 -p 16 -c petalinux-2019.1-docker.tar > petalinux-2019.1-docker.tar.gz
    ```
    or
    ```bash
    docker save petalinux:2019.1 | pigz -9 -p 16 > petalinux-2019.1-docker.tar.gz
    ```

### konosubakonoakua · 2024-09-25T13:49:53Z

## nfs setup
Config nfs (optional, used for Linux rootfs mount).
```bash
sudo mkdir -p /nfsroot
sudo chown -R rf:rf /nfsroot
```
> [!CAUTION]
> `/nfsroot` is very similar with `/nfsboot`, never make a typo like this!!! 🚨🚨🚨😱😱😱
```bash
sudo cp -f /etc/exports /etc/exports.bak
sudo tee -a /etc/exports << EOF
/nfsroot *(rw,sync,no_root_squash)
EOF
```
if encountered: `VFS:Unable to mount root fs via NFS, tring floppy`:
```bash
sudo cp -f /etc/default/nfs-kernel-server /etc/default/nfs-kernel-server.bak
sudo tee -a /etc/default/nfs-kernel-server << EOF
RPCNFSDOPTS="--nfs-version 2,3,4 --debug --syslog"
EOF
sudo systemctl restart nfs-kernel-server
```
restart after modifications.
```bash
sudo service nfs-kernel-server restart
```
show shared folders
```bash
showmount -e
```
refresh on the fly
```bash
sudo exportfs -rv
```
test nfs
```bash
sudo mkdir -p /mnt/nfs
sudo mount -t nfs 127.0.0.1:/nfsroot /mnt/nfs
```
### references
- https://zhuanlan.zhihu.com/p/421380346

### konosubakonoakua · 2024-09-25T21:51:45Z

## references
- 

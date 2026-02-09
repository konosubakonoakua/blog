---
number: 92
title: "[sw][linux] ubuntu 22 settings"
state: open
url: https://github.com/konosubakonoakua/blog/issues/92
author: konosubakonoakua
createdAt: 2024-09-23T03:39:47Z
updatedAt: 2025-07-11T07:19:44Z
labels: ["Linux"]
---

# 92 [sw][linux] ubuntu 22 settings

## disable/enable wayland
```shell
# disable wayland
sudo sed -i "s/#WaylandEnable/WaylandEnable/" /etc/gdm3/custom.conf
# enable wayland
sudo sed -i "s/WaylandEnable/#WaylandEnable/" /etc/gdm3/custom.conf
# or just manually
sudo vim /etc/gdm3/custom.conf
```

---

## Comments

### konosubakonoakua · 2024-09-23T06:17:30Z

## install a better file explorer
```bash
sudo apt-get install ?
```

### konosubakonoakua · 2024-11-12T17:18:23Z

## mirrors (multi-arch)
- https://wiki.debian.org/Multiarch/HOWTO
```bash
sudo dpkg --add-architecture i386
sudo dpkg --add-architecture armhf
sudo dpkg --add-architecture arm64
sudo dpkg --print-foreign-architectures
```
```bash
sudo cp -f /etc/apt/sources.list /etc/apt/sources.list.bak
```
```bash
sudo tee /etc/apt/sources.list << 'EOF'

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb [arch=amd64,i386] https://mirrors.bfsu.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb [arch=amd64,i386] https://mirrors.bfsu.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb [arch=amd64,i386] https://mirrors.bfsu.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.bfsu.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.bfsu.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.bfsu.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb [arch=amd64,i386] http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.bfsu.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.bfsu.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse

######################################################################################################

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb [arch=arm64,armhf] https://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb [arch=arm64,armhf] https://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb [arch=arm64,armhf] https://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
# deb-src https://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb [arch=arm64,armhf] http://ports.ubuntu.com/ubuntu-ports/ jammy-security main restricted universe multiverse
# deb-src http://ports.ubuntu.com/ubuntu-ports/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb [arch=armel,armhf] https://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse

EOF
```

### konosubakonoakua · 2024-12-13T03:47:26Z

## root passwd reset (`/etc/shadow`)
```bash
openssl passwd -1 "your-password-here"
```
copy generated password to `/etc/shadow`:
```
# cat /etc/shadow
root:<your password>:10933:0:99999:7:::
```


### konosubakonoakua · 2024-12-13T03:52:15Z

## avahi-daemon (mdns)
```bash
hostname <your hostname>
sudo vi /etc/hostname
```
```bash
sudo apt install avahi-daemon
sudo systemctl start avahi-daemon
sudo systemctl enable avahi-daemon
```

### konosubakonoakua · 2024-12-13T03:53:56Z

## ntp
### systemd-timesyncd
```bash
timedatectl set-timezone Asia/Shanghai
sudo date --set="2025-02-12 11:32:00"
```
```bash
sudo apt -y install systemd-timesyncd
sudo systemctl enable systemd-timesyncd
sudo systemctl start systemd-timesyncd
```

### konosubakonoakua · 2025-07-11T07:19:44Z

## animations
```bash
gsettings set org.gnome.desktop.interface enable-animations false # or true
```

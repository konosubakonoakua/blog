---
number: 78
title: "[sw][linux][raspi][ioc] Raspberry Pi setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/78
author: konosubakonoakua
createdAt: 2024-08-03T18:41:30Z
updatedAt: 2026-01-21T03:54:53Z
labels: ["Linux", "EPICS"]
---

# 78 [sw][linux][raspi][ioc] Raspberry Pi setup

### raspberry-pi-imager
<details>

```powershell
scoop install raspberry-pi-imager
```
Select board type & os, then flash sd card.

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
sudo smbpasswd -a rpi
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

### Install fzf/zellij/zoxide

<details>

Download them from windows side
- [zellij-aarch64-unknown-linux-musl.tar.gz](https://github.com/zellij-org/zellij/releases)
- [fzf](https://github.com/junegunn/fzf/releases)
- [zoxide](https://github.com/ajeetdsouza/zoxide/releases)
- [neovim]()

[share-files-with-scp](https://www.raspberrypi.com/documentation/computers/remote-access.html#share-files-with-scp)
```bash
scp fzf zellij zoxide rpi@impcas-ioc-rpi4-001.local:shared/
```
Or just use samba on windows side.

### Install nvim
```bash
sudo apt update
sudo apt install snapd
sudo reboot
```
```bash
sudo snap install snapd
sudo snap install --classic nvim
```

</details>

### rc-local

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

### uv & python

<details>

```bash
sudo apt-get install libffi-dev libjpeg-dev zlib1g-dev

```
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
uv python list
```
```bash
uv python install cpython-3.13.0+freethreaded-linux-aarch64-gnu
```
```bash
uv python install cpython-3.12.7-linux-aarch64-gnu
```
```bash
uv python install cpython-3.11.10-linux-aarch64-gnu
```
```bash
cat > ~/.config/uv/uv.toml << 'EOF'
index-url = "https://mirrors.bfsu.edu.cn/pypi/web/simple"
EOF
```

</details>

### rust
<details>

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

</details>



---

## Comments

### konosubakonoakua · 2024-11-21T02:15:45Z

### ssh
#123

### konosubakonoakua · 2024-11-25T07:46:44Z

### nodejs
#### use `nvm`
<details>

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```
Install nodejs 20 version
```bash
nvm install 20
nvm use 20
```

</details>

### konosubakonoakua · 2024-11-26T02:39:23Z

### docker
<details>

- https://docs.docker.com/engine/install/debian/
- https://github.com/konosubakonoakua/blog/issues/84#issuecomment-2499518392

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
```

```bash
case $(uname -a) in
  *ubuntu*) HOST_OS=ubuntu;;
  *Debian*) HOST_OS=debian;;
  *)
    echo "Unsupported system"
    exit 1
    ;;
esac
```

```bash
# Add Docker's official GPG key:
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/${HOST_OS}/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/${HOST_OS} \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo docker run hello-world
```

</details>

### konosubakonoakua · 2024-11-26T03:25:11Z

## Network
#### ~enable ipv4 on raspberrypi 3b with networkmanager (without raspi offical os)~

<details>

- [x] use `nmtui-edit` to modify `Wired Connection 1` located @ `system-connections/Wired\ connection\ 1.nmconnection `

    ```bash
    rpi@<hidden>:/etc/NetworkManager $ sudo cat system-connections/Wired\ connection\ 1.nmconnection 

    [connection]
    id=Wired connection 1
    uuid=<hidden>
    type=ethernet
    autoconnect-priority=-999
    interface-name=<hidden>
    
    [ethernet]
    
    [ipv4]
    address1=100.100.100.100/8,100.100.100.254
    may-fail=false
    method=manual
    
    [ipv6]
    addr-gen-mode=default
    method=disabled
    
    [proxy]
    
    ```
- [ ] but how to automatically disable ipv6 when flashing os into sdcard?
    ```bash
    rpi@<hidden>:/etc/NetworkManager/system-connections $ sudo cat preconfigured.nmconnection 
    [connection]
    id=preconfigured
    uuid=<hidden>
    type=wifi
    [wifi]
    mode=infrastructure
    ssid=rfhjj
    hidden=false
    [ipv4]
    method=auto
    [ipv6]
    addr-gen-mode=default
    method=auto
    [proxy]
    [wifi-security]
    key-mgmt=wpa-psk
    psk=<hidden>
    ```
- [x] how to make mdns working
  - We need setup ipv4 first, use `nmtui-edit` then disable then enable connection by `nmtui`
  - https://wiki.archlinux.org/title/Avahi
  - In case you want to change mdns domain & host: `sudo vi /etc/avahi/avahi-daemon.conf`
    ```bash
    [server]
    host-name=<your host name>
    domain-name=local
    #browse-domains=0pointer.de, zeroconf.org
    use-ipv4=yes
    use-ipv6=no
    allow-interfaces=eth0
    #deny-interfaces=eth1
    #check-response-ttl=no
    #use-iff-running=no
    #enable-dbus=yes
    #disallow-other-stacks=no
    #allow-point-to-point=no
    #cache-entries-max=4096
    #clients-max=4096
    #objects-per-client-max=1024
    #entries-per-entry-group-max=32
    ratelimit-interval-usec=1000000
    ratelimit-burst=1000
    ```

</details>

### konosubakonoakua · 2024-11-27T12:31:21Z

### epics on raspi
<details>

- [epics 7 installation](https://sfeister.github.io/sidekick-epics-docs/model1/pi-epics-install/)
- [rpi epics](https://cmd-response.readthedocs.io/en/latest/epics/rpi_epics.html)
```bash
sudo apt -y install git net-tools build-essential re2c libnet-dev libpcap-dev libusb-1.0-0-dev
```
```bash
mkdir $HOME/EPICS
cd $HOME/EPICS
##git clone --recursive https://github.com/epics-base/epics-base.git
wget https://epics.anl.gov/download/base/base-7.0.8.1.tar.gz
tar xf base-7.0.8.1.tar.gz && ln -s base-7.0.8.1 epics-base
cd epics-base
make -j2 # rpi4b
~/EPICS/bin/linux-aarch64/caget
```
> [!IMPORTANT]
> You need to set `PYEPICS_LIBCA` to load the correct `libca.so` on rpi
> Refer to: [pyepics issue 238](https://github.com/pyepics/pyepics/issues/238)
```bash
cat >> ~/.bashrc << 'EOF'

export PYEPICS_LIBCA=$HOME/EPICS/epics-base/lib/linux-aarch64/libca.so

EOF
```
#### pythonsoftioc on raspi
```toml
# ~/.config/uv/uv.toml   (Linux/macOS)
[extra-build-dependencies]
softioc = [{ requirement = "epicscorelibs", match-runtime = true }]
```
```bash
uv pip install pvxslibs epicsdbbuilder pyyaml cothread
uv pip install softioc --no-deps
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
python -c "import pcaspy"
```

If encountered with: `ImportError: libca.so.4.14.3: cannot open shared object file: No such file or directory`, Check version first:

```bash
caget -V
```
```bash
ldd ~/.virtualenvs/ioc/lib/python3.12/site-packages/pcaspy/_cas.cpython-312-aarch64-linux-gnu.so
```
If epics version not matched with pcas build, then use `ln -s` to make softlinks for shared libs.

```bash
cd $EPICS_BASE/lib/linux-aarch64/
ln -s libca.so.4.14.4 libca.so.4.14.3
ln -s libCom.so.3.23.1 libCom.so.3.23.0
```

If encountered with: `ImportError: libdbCore.so.3.23.0: cannot open shared object file: No such file or directory`, need to reinstall pcaspy using pip:

```bash
 uv pip install --reinstall pcaspy  --no-cache
```


</details>

### konosubakonoakua · 2024-11-27T13:19:33Z

## gpio libs
<details>

<summary>pinout</summary>

![https://pinout.xyz/](../assets/78/1.png)

</details>

```bash
uv pip install adafruit-blinka gpiozero smbus smbus2 smbus3 spidev pyserial lgpio
```

### konosubakonoakua · 2024-11-28T14:07:12Z

## serial port monitoring
```bash
sudo apt install tio
tio -a latest --exclude-drivers=!(cp210x)
```

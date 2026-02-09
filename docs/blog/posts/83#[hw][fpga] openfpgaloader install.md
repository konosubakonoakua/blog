---
number: 83
title: "[hw][fpga] openfpgaloader install"
state: open
url: https://github.com/konosubakonoakua/blog/issues/83
author: konosubakonoakua
createdAt: 2024-09-20T01:10:43Z
updatedAt: 2024-10-17T06:39:41Z
labels: ["FPGA", "Linux", "Hardware"]
---

# 83 [hw][fpga] openfpgaloader install

## installation
- https://trabucayre.github.io/openFPGALoader/guide/install.html

```bash
sudo apt install \
  git \
  gzip \
  libftdi1-2 \
  libftdi1-dev \
  libhidapi-hidraw0 \
  libhidapi-dev \
  libudev-dev \
  zlib1g-dev \
  cmake \
  pkg-config \
  make \
  g++
```

```bash
git clone https://github.com/trabucayre/openFPGALoader
cd openFPGALoader
mkdir build
cd build
cmake .. # add -DBUILD_STATIC=ON to build a static version
         # add -DENABLE_UDEV=OFF to disable udev support and -d /dev/xxx
         # add -DENABLE_CMSISDAP=OFF to disable CMSIS DAP support
cmake --build .
# or
make -j$(nproc)
sudo make install
cd ..
```
```bash
sudo cp 99-openfpgaloader.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger # force udev to take new rule
sudo usermod -a $USER -G plugdev # add user to plugdev group
```

---

## Comments

### konosubakonoakua · 2024-09-20T01:33:10Z

## download (much faster than vivado)
### to flash
```bash
openFPGALoader -c digilent_hs2 --fpga-part xc6slx25csg324 -f ./sp67015dog.mcs --verbose-level 2
```

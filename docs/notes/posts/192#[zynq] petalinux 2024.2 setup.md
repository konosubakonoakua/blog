---
number: 192
title: "[zynq] petalinux 2024.2 setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/192
author: konosubakonoakua
createdAt: 2025-12-17T06:41:53Z
updatedAt: 2025-12-27T03:30:39Z
labels: []
---

# 192 [zynq] petalinux 2024.2 setup

## Dependencies

```bash
sudo apt install \
  gawk \
  gcc \
  net-tools \
  libncurses5-dev \
  zlib1g-dev \
  libssl-dev \
  libselinux1 \
  xterm \
  gcc-multilib \
  build-essential \
  automake \
  screen \
  g++ \
  python3-pip \
  xz-utils \
  cpp \
  patch \
  python3-git \
  python3-jinja2 \
  python3-pexpect \
  diffutils \
  debianutils \
  iputils-ping \
  libegl1-mesa \
  libsdl1.2-dev \
  pylint \
  cpio \
  gnupg \
  dnsutils \
  libtool \
  texinfo \
  libtinfo5 \
  cifs-utils
```

---

## Comments

### konosubakonoakua · 2025-12-17T08:23:38Z

## sstate cache
- <plnx-proj-root>/project-spec/meta-user/conf/petalinuxbsp.conf

  ```bash
  DL_DIR = "/var/cache/petalinux-sstate/2024.2/downloads"
  SSTATE_DIR = "/var/cache/petalinux-sstate/2024.2/aarch64"
  ```
  ```bash
  sudo mkdir -p /var/cache/petalinux-sstate/
  sudo chown impcas:impcas /var/cache/petalinux-sstate/
  mkdir -p /var/cache/petalinux-sstate/2024.2/downloads
  mkdir -p /var/cache/petalinux-sstate/2024.2/aarch64
  mkdir -p /var/cache/petalinux-sstate/2024.2/arm
  ```
- yocto settings
  ```bash
  $ petalinux-config ---> Yocto Settings ---> Add pre-mirror url ---> file:///mnt/petalinux-sstate/2024.2/downloads
  $ petalinux-config ---> Yocto Settings ---> Local sstate feeds settings ---> local sstate feeds url ---> /mnt/petalinux-sstate/2024.2/aarch64
  ```
- samba share
  ```bash
  sudo mount -t cifs //rf-pc-kuro.local/share/petalinux/sstate /mnt/petalinux-sstate -o credentials=~/.rf-pc-kuro.samba.cred
  sudo mount -t cifs //rf-pc-kuro.local/share/petalinux/sstate /mnt/petalinux-sstate -o username="",password=""
  ```
- logging
  ```log
  Parsing recipes: 100% |################################################################################################################################| Time: 0:05:05
  Parsing of 5800 .bb files complete (0 cached, 5800 parsed). 8454 targets, 994 skipped, 27 masked, 0 errors.
  NOTE: Resolving any missing task queue dependencies
  NOTE: Fetching uninative binary shim file:///home/impcas/impcas/impcas_hw_fpga_zu9_viv24_plnx24/petalinux/components/yocto/downloads/uninative/6bf00154c5a7bc48adbf63f
  d17684bb87eb07f4814fbb482a3fbd817c1ccf4c5/x86_64-nativesdk-libc-4.6.tar.xz;sha256sum=6bf00154c5a7bc48adbf63fd17684bb87eb07f4814fbb482a3fbd817c1ccf4c5 (will check PREM
  IRRORS first)
  Initialising tasks:  57% |########################################################################                                                     | ETA:  0:00:24
  ```

### konosubakonoakua · 2025-12-17T16:12:16Z

## petalinux-build
### memory 16GB & cpu 4 cores
- not working

  ```bash
  $ petalinux-config ---> Yocto Settings ---> Parallel thread execution ---> (10) sets number of bb threads (BB_NUMBER_THREADS)
  $ petalinux-config ---> Yocto Settings ---> Parallel thread execution ---> (10) sets number of parallel make -j threads (PARALLEL_MAKE)
  ```
- os limit

  ```bash
  taskset -c 0,1,2,3 petalinux-build
  ```

### konosubakonoakua · 2025-12-18T01:22:28Z

## Issue
### mpmu-rom-native-1.0-r0 do_fetch: Bitbake Fetcher Error

```log
The bitbake file has a SRC_URI = "https://www.xilinx.com/bin/public/openDownload?filename=PMU_ROM.tar.gz", which when using a local cache gets converted into looking for the file "[openDownload?filename=PMU_ROM.tar.gz](https://www.xilinx.com/bin/public/openDownload?filename=PMU_ROM.tar.gz)" on the disk. This is fine in Linux, but in Windows a '?' character in the filename is forbidden. As I was using Windows tools to extract the files from the download archive, this particular file was being renamed.
**My fix was to use Linux to create these files from the archive.**
```

```bb
# "petalinux/components/yocto/layers/meta-xilinx/meta-xilinx-core/recipes-bsp/pmu-firmware/pmu-rom-native.bb" line 10
SRC_URI = "https://www.xilinx.com/bin/public/openDownload_filename=PMU_ROM.tar.gz"  
```

### armv-pmu: probe of pmu failed with error -22
If you forgot to include bitstream to BOOT.BIN, you will get this.
```log
[    1.630546] armv8-pmu pmu: hw perfevents: failed to parse interrupt-affinity[2]
[    1.639463] armv8-pmu: probe of pmu failed with error -22
[    1.645586] fpga_manager fpga0: Xilinx ZynqMP FPGA Manager registered
[    1.651641] usbcore: registered new interface driver snd-usb-audio
[    1.658101] pktgen: Packet Generator for packet performance testing. Version: 2.75
[    1.666321] Initializing XFRM netlink socket
[    1.669324] NET: Registered PF_INET6 protocol family
[    1.674860] Segment Routing with IPv6
[    1.677912] In-situ OAM (IOAM) with IPv6
[    1.681884] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    1.688078] NET: Registered PF_PACKET protocol family
[    1.692779] NET: Registered PF_KEY protocol family
[    1.697567] can: controller area network core
[    1.701932] NET: Registered PF_CAN protocol family
[    1.706693] can: raw protocol
[    1.709646] can: broadcast manager protocol
[    1.713830] can: netlink gateway - max_hops=1
[    1.718258] Bluetooth: RFCOMM TTY layer initialized
[    1.723053] Bluetooth: RFCOMM socket layer initialized
[    1.728191] Bluetooth: RFCOMM ver 1.11
[    1.731925] Bluetooth: BNEP (Ethernet Emulation) ver 1.3
[    1.737227] Bluetooth: BNEP filters: protocol multicast
[    1.742449] Bluetooth: BNEP socket layer initialized
[    1.747407] Bluetooth: HIDP (Human Interface Emulation) ver 1.2
[    1.753325] Bluetooth: HIDP socket layer initialized
[    1.758320] 8021q: 802.1Q VLAN Support v1.8
[    1.762590] 9pnet: Installing 9P2000 support
[    1.766751] Key type dns_resolver registered
[    1.771087] NET: Registered PF_VSOCK protocol family
[    1.783689] registered taskstats version 1
[    1.783729] Loading compiled-in X.509 certificates
[    1.794974] Btrfs loaded, zoned=no, fsverity=no
[    1.795146] alg: No test for xilinx-zynqmp-rsa (zynqmp-rsa)
[    2.437121] ff000000.serial: ttyPS0 at MMIO 0xff000000 (irq = 23, base_baud = 6249999) is a xuartps
[    2.446202] printk: console [ttyPS0] enabled
[    2.446202] printk: console [ttyPS0] enabled
[    2.450500] printk: bootconsole [cdns0] disabled
[    2.450500] printk: bootconsole [cdns0] disabled
[   23.470487] rcu: INFO: rcu_sched detected stalls on CPUs/tasks:
[   23.476398] rcu:     1-...0: (1 GPs behind) idle=171c/1/0x4000000000000000 softirq=83/83 fqs=2625
[   23.484998] rcu:     (detected by 0, t=5252 jiffies, g=-1031, q=9 ncpus=2)
[   23.491605] Task dump for CPU 1:
[   23.494815] task:kworker/u6:1    state:R  running task     stack:0     pid:49    ppid:2      flags:0x0000000a
[   23.504720] Workqueue: events_unbound deferred_probe_work_func
[   23.510553] Call trace:
[   23.512983]  __switch_to+0xdc/0x154
[   23.516463]  0xffff8000816e9c90
```
### CPU hangs when using devmem/userspace drivers in Linux
- https://adaptivesupport.amd.com/s/article/000036858?language=en_US

### konosubakonoakua · 2025-12-25T08:36:47Z

## Tips
### pl device tree node
```bash
root@plnx:~# ls /proc/device-tree/pl-bus/
'#address-cells'   compatible      led_blinker_axion_reg@80030000   phandle            ranges
'#size-cells'      gpio@80020000   name                             pl_regs@80000000
```

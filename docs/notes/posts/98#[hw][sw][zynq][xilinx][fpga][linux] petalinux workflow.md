---
number: 98
title: "[hw][sw][zynq][xilinx][fpga][linux] petalinux workflow"
state: open
url: https://github.com/konosubakonoakua/blog/issues/98
author: konosubakonoakua
createdAt: 2024-09-23T23:14:55Z
updatedAt: 2025-03-31T03:10:28Z
labels: ["FPGA", "ZYNQ", "Linux"]
---

# 98 [hw][sw][zynq][xilinx][fpga][linux] petalinux workflow

## design flow overview 
- https://github.com/gracepua/PetaLinux-Guide
- https://github.com/DmitryAndSoCs/Cora_Z7_RT_Linux_example/blob/main/docs/src/petalinux_project_guide.md
<img src="https://github.com/user-attachments/assets/463896e0-8391-4c8e-b83e-cd4dfed8e104" width="60%" />

## design flow overview (text version)
```
petalinux-create -t <project name>                                  //创建 petalinux 工程
petalinux-config --get-hw-description <path to hdf or xsa file>     //导入 hdf/xsa 硬件配置文件
petalinux-config                                                    //打开 petalinux 工程的图形化配置页面
petalinux-config -c kernel                                          //定制 linux 内核
petalinux-config -c rootfs                                          //定制根文件系统
petalinux-build                                                     //编译整个 petalinux 工程
petalinux-package --boot --fsbl --fpga --u-boot --force             //将--后面的文件全部打包生成 BOOT.BIN 文件
pelinux-boot jtag --prebuilt 2                                      //使用 jtag 下载，并启动
```

---

## Comments

### konosubakonoakua · 2024-09-24T19:00:38Z

## create project
```bash
petalinux-create -h
```
## config petalinux
```bash
petalinux-config --get-hw-description=<xsa folder>
```
## sstate
```bash
_PETA_ARCH=arm
#_PETA_ARCH=aarch64
_PETA_VERSION=2019.1
#_PETA_VERSION=2024.1
_CONFIG_FILE=project-spec/configs/config
cp $_CONFIG_FILE $_CONFIG_FILE.bak
sed -i -e "s/^\(CONFIG_YOCTO_NETWORK_SSTATE_FEEDS\)=.*/\1=n/" \
       -e "s#^\(CONFIG_PRE_MIRROR_URL\)=.*#\1=\"file:///tools/petalinux/sstate/$_PETA_VERSION/downloads\"#" \
       -e "s#^\(CONFIG_YOCTO_LOCAL_SSTATE_FEEDS_URL\)=.*#\1=\"/tools/petalinux/sstate/$_PETA_VERSION/$_PETA_ARCH\"#" \
       -e "s/^# CONFIG_YOCTO_BB_NO_NETWORK is not set/CONFIG_YOCTO_BB_NO_NETWORK=y/" \
       $_CONFIG_FILE

```
```bash
petalinux-config --silentconfig
```

## config kernel
```bash
petalinux-config -c kernel
```
## config rootfs
```bash
petalinux-config -c rootfs
```
## build
```bash
petalinux-build
```
## make BOOT.bin
```bash
petalinux-package --boot --fsbl --fpga --u-boot --force
```
## make prebuilt
```bash
petalinux-package --prebuilt
```
## boot
```bash
pelinux-boot --jtag --prebuilt 2
```
> [!IMPORTANT]
> If you encoutered the following error, JUST
> *Smaaaaaashhhhhhhhhhh* the reset button on your board. 🙄🙄🙄
> or do some gentle but similar 🧐
<details>
<summary>[ERROR] Command 'xsdb /tmp/tmpsetc21xx' returned non-zero exit status 1.</summary>

```log
rf@myPC:~ $ petalinux-boot jtag --prebuilt 2
[WARNING] Will not program bitstream on the target. If you want to program bitstream,
[WARNING] Use --fpga <BITSTREAM> option to specify a bitstream.
[INFO] Launching XSDB for file download and boot.
[INFO] This may take a few minutes, depending on the size of your image.
attempting to launch hw_server

****** Xilinx hw_server v2024.1
  **** Build date : May 16 2024 at 12:16:21
    ** Copyright 1986-2022 Xilinx, Inc. All Rights Reserved.
    ** Copyright 2022-2024 Advanced Micro Devices, Inc. All Rights Reserved.

INFO: hw_server application started
INFO: Use Ctrl-C to exit hw_server application

INFO: To connect to this hw_server instance use url: TCP:127.0.0.1:3121

Memory read error at 0xF8007080. MMU section translation fault
    invoked from within
"::tcf::eval -progress {apply {{msg} {puts $msg}}} {tcf_send_command tcfchan#0 Memory get siiii {Bea{o{msg o{} A}}} {JTAG-jsn-JTAG-SMT2-210251190311-4ba00477-0.0 4160778368 4 4 7}}"
    (procedure "::tcf::send_command" line 4)
    invoked from within
"::tcf::send_command $chan Memory get "siiii" "Bea{o{msg o{} A}}" [list $ctx $start_addr $size $nbytes $mode]"
    (procedure "mrd" line 87)
    invoked from within
"mrd 0xF8007080"
    (procedure "ps_version" line 2)
    invoked from within
"ps_version"
    (procedure "ps7_post_config" line 8)
    invoked from within
"ps7_post_config"
    (file "/tmp/tmpsetc21xx" line 8)
[ERROR] Command 'xsdb /tmp/tmpsetc21xx' returned non-zero exit status 1.
```

</details>


### konosubakonoakua · 2024-09-25T17:31:15Z

## 



### konosubakonoakua · 2024-09-25T17:50:07Z

## nfs boot
### config nfs server
Refer to https://github.com/konosubakonoakua/blog/issues/97#issuecomment-2374149784
### config petalinux
```bash
petalinux-config
```
<img src="https://github.com/user-attachments/assets/202aeca8-2aa1-4ca9-ba8d-b0a2c5e88406" width="80%" />

<img src="https://github.com/user-attachments/assets/14bbbdfc-ca33-49a9-9e88-eef570cda605" width="80%" />

<img src="https://github.com/user-attachments/assets/5ab7be9d-3555-46db-bb6d-4262208e7f81" width="80%" />

### overwrite bootargs in devicetree
> [!TIP]
> bootargs generated by `petalinux-cofing` doesn't have correct ip config (wrong gateway & netmask)
add node below to `project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi`
> OR, maybe I messed up something.
```python
	chosen {
		//bootargs = "console=ttyPS0,115200 earlycon root=/dev/nfs rootfstype=nfs nfsroot=10.9.8.8:/nfsroot,tcp,nfsvers=3 ip=dhcp rw";
		bootargs = "console=ttyPS0,115200 earlycon root=/dev/nfs rootfstype=nfs nfsroot=10.9.8.8:/nfsroot,tcp,nfsvers=3 ip=10.9.8.9:10.9.8.8:10.9.8.254:255.255.255.0::eth0:off rw";
		stdout-path = "serial0:115200n8";
	};
```
### build petalinux
```bash
petalinux-build
```
### package prebuilt petalinux
```bash
petalinux-package prebuilt --force
```
### remove `INITRD` from `pxelinux.cfg/default`
```bash
PXE_CFG_FILE=/tftpboot/pxelinux.cfg/default
cp -f $PXE_CFG_FILE $PXE_CFG_FILE.bak
sed -i '/^INITRD/d' $PXE_CFG_FILE
```

### boot by jtag
```bash
petalinux-boot jtag --prebuilt 2
```
### or boot by sdcard
```bash
TFTPBOOT_PATH=/tftpboot
SDBOOT_PATH=/media/$USER/BOOT
cd $TFTPBOOT_PATH
cp BOOT.BIN boot.scr image.ub system.dtb $SDBOOT_PATH
ll $SDBOOT_PATH
umount /media/$USER/BOOT /media/$USER/rootfs
# or jsut umount /media/$USER/*
cd -
```
### observe on serial
```bash
tio -a latest --exclude-drivers=!(cp210x)
```
### check `bootarg` or `cmdline` after login
```bash
cat /proc/cmdline
```

### konosubakonoakua · 2024-09-25T21:54:14Z

### nfs rootfs boot log
<details>
<summary>nfs_rootfs_boot.log</summary>

```shell
$ tio -a latest --exclude-drivers=!(cp210x)
[06:36:15.541] tio v3.7
[06:36:15.541] Press ctrl-t q to quit
[06:36:15.605] Warning: Could not open (null) (Bad address)
[06:36:15.605] Waiting for tty device..
[06:36:32.642] Connected to /dev/ttyUSB0
 0
switch to partitions #0, OK
mmc0 is current device
Scanning mmc 0:1...
Found U-Boot script /boot.scr
3473 bytes read in 13 ms (260.7 KiB/s)
## Executing script at 03000000
Trying to load boot images from mmc0
4516312 bytes read in 258 ms (16.7 MiB/s)
## Loading kernel from FIT Image at 10000000 ...
   Using 'conf-system-top.dtb' configuration
   Verifying Hash Integrity ... OK
   Trying 'kernel-1' kernel subimage
     Description:  Linux kernel
     Type:         Kernel Image
     Compression:  uncompressed
     Data Start:   0x10000110
     Data Size:    4496024 Bytes = 4.3 MiB
     Architecture: ARM
     OS:           Linux
     Load Address: 0x00200000
     Entry Point:  0x00200000
     Hash algo:    sha256
     Hash value:   d2a41d8c8fd05474c0426587c386c134138d364722a1a3bdd079ea2911452c66
   Verifying Hash Integrity ... sha256+ OK
## Loading fdt from FIT Image at 10000000 ...
   Using 'conf-system-top.dtb' configuration
   Verifying Hash Integrity ... OK
   Trying 'fdt-system-top.dtb' fdt subimage
     Description:  Flattened Device Tree blob
     Type:         Flat Device Tree
     Compression:  uncompressed
     Data Start:   0x10449cb4
     Data Size:    18367 Bytes = 17.9 KiB
     Architecture: ARM
     Hash algo:    sha256
     Hash value:   fb2e10d77acfb07471d3e3a77ae4f7d62bcd34ead5d6c8fed2a220ebd8c98abc
   Verifying Hash Integrity ... sha256+ OK
   Booting using the fdt blob at 0x10449cb4
Working FDT set to 10449cb4
   Loading Kernel Image
   Loading Device Tree to 2fff8000, end 2ffff7be ... OK
Working FDT set to 2fff8000
                                                                                                                                                                         
Starting kernel ...
                                                                                                                                                                         
Booting Linux on physical CPU 0x0
Linux version 6.6.10-xilinx-v2024.1-g3af4295e00ef (oe-user@oe-host) (arm-xilinx-linux-gnueabi-gcc (GCC) 12.2.0, GNU ld (GNU Binutils) 2.39.0.20220819) #1 SMP PREEMPT Sat Apr 27 05:22:24 UTC 2024
CPU: ARMv7 Processor [413fc090] revision 0 (ARMv7), cr=18c5387d
CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
OF: fdt: Machine model: xlnx,zynq-7000
earlycon: cdns0 at MMIO 0xe0001000 (options '115200n8')
printk: bootconsole [cdns0] enabled
Memory policy: Data cache writealloc
cma: Reserved 16 MiB at 0x3f000000 on node -1
Zone ranges:
  Normal   [mem 0x0000000000000000-0x000000002fffffff]
  HighMem  [mem 0x0000000030000000-0x000000003fffffff]
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000000000000-0x000000003fffffff]
Initmem setup node 0 [mem 0x0000000000000000-0x000000003fffffff]
percpu: Embedded 12 pages/cpu s17940 r8192 d23020 u49152
Kernel command line: console=ttyPS0,115200 earlycon root=/dev/nfs rootfstype=nfs nfsroot=10.9.8.8:/nfsroot,tcp,nfsvers=3 ip=10.9.8.9:10.9.8.8:10.9.8.254:255.255.255.0::eth0:off rw
Dentry cache hash table entries: 131072 (order: 7, 524288 bytes, linear)
Inode-cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
Built 1 zonelists, mobility grouping on.  Total pages: 260608
mem auto-init: stack:all(zero), heap alloc:off, heap free:off
Memory: 1011916K/1048576K available (6144K kernel code, 229K rwdata, 1920K rodata, 1024K init, 146K bss, 20276K reserved, 16384K cma-reserved, 245760K highmem)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=2, Nodes=1
rcu: Preemptible hierarchical RCU implementation.
rcu:    RCU event tracing is enabled.
rcu:    RCU restricting CPUs from NR_CPUS=4 to nr_cpu_ids=2.
rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=2
NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
efuse mapped to (ptrval)
slcr mapped to (ptrval)
GIC physical location is 0xf8f01000
L2C: platform modifies aux control register: 0x72360000 -> 0x72760000
L2C: DT/platform modifies aux control register: 0x72360000 -> 0x72760000
L2C-310 erratum 769419 enabled
L2C-310 enabling early BRESP for Cortex-A9
L2C-310 full line of zeros enabled for Cortex-A9
L2C-310 ID prefetch enabled, offset 1 lines
L2C-310 dynamic clock gating enabled, standby mode enabled
L2C-310 cache controller enabled, 8 ways, 512 kB
L2C-310: CACHE_ID 0x410000c8, AUX_CTRL 0x76760001
rcu: srcu_init: Setting srcu_struct sizes based on contention.
zynq_clock_init: clkc starts at (ptrval)
Zynq clock init
sched_clock: 64 bits at 167MHz, resolution 6ns, wraps every 4398046511103ns
clocksource: arm_global_timer: mask: 0xffffffffffffffff max_cycles: 0x26703d7dd8, max_idle_ns: 440795208065 ns
Switching to timer-based delay loop, resolution 6ns
Console: colour dummy device 80x30
Calibrating delay loop (skipped), value calculated using timer frequency.. 333.33 BogoMIPS (lpj=1666666)
CPU: Testing write buffer coherency: ok
CPU0: Spectre v2: using BPIALL workaround
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 2048 (order: 1, 8192 bytes, linear)
Mountpoint-cache hash table entries: 2048 (order: 1, 8192 bytes, linear)
CPU0: thread -1, cpu 0, socket 0, mpidr 80000000
Setting up static identity map for 0x100000 - 0x100060
rcu: Hierarchical SRCU implementation.
rcu:    Max phase no-delay instances is 1000.
smp: Bringing up secondary CPUs ...
CPU1: thread -1, cpu 1, socket 0, mpidr 80000001
CPU1: Spectre v2: using BPIALL workaround
smp: Brought up 1 node, 2 CPUs
SMP: Total of 2 processors activated (666.66 BogoMIPS).
CPU: All CPU(s) started in SVC mode.
devtmpfs: initialized
VFP support v0.3: implementor 41 architecture 3 part 30 variant 9 rev 4
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
futex hash table entries: 512 (order: 3, 32768 bytes, linear)
pinctrl core: initialized pinctrl subsystem
NET: Registered PF_NETLINK/PF_ROUTE protocol family
DMA: preallocated 256 KiB pool for atomic coherent allocations
thermal_sys: Registered thermal governor 'step_wise'
cpuidle: using governor menu
platform axi: Fixed dependency cycle(s) with /axi/interrupt-controller@f8f01000
amba f8801000.etb: Fixed dependency cycle(s) with /replicator/out-ports/port@1/endpoint
amba f8803000.tpiu: Fixed dependency cycle(s) with /replicator/out-ports/port@0/endpoint
amba f8804000.funnel: Fixed dependency cycle(s) with /replicator/in-ports/port/endpoint
amba f889c000.ptm: Fixed dependency cycle(s) with /axi/funnel@f8804000/in-ports/port@0/endpoint
amba f889d000.ptm: Fixed dependency cycle(s) with /axi/funnel@f8804000/in-ports/port@1/endpoint
hw-breakpoint: found 5 (+1 reserved) breakpoint and 1 watchpoint registers.
hw-breakpoint: maximum watchpoint size is 4 bytes.
e0001000.serial: ttyPS0 at MMIO 0xe0001000 (irq = 26, base_baud = 6249999) is a xuartps
printk: console [ttyPS0] enabled
printk: console [ttyPS0] enabled
printk: bootconsole [cdns0] disabled
printk: bootconsole [cdns0] disabled
SCSI subsystem initialized
mc: Linux media interface: v0.10
videodev: Linux video capture interface: v2.00
pps_core: LinuxPPS API ver. 1 registered
pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
PTP clock support registered
EDAC MC: Ver: 3.0.0
FPGA manager framework
clocksource: Switched to clocksource arm_global_timer
NET: Registered PF_INET protocol family
IP idents hash table entries: 16384 (order: 5, 131072 bytes, linear)
tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 4096 bytes, linear)
Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
TCP established hash table entries: 8192 (order: 3, 32768 bytes, linear)
TCP bind hash table entries: 8192 (order: 5, 131072 bytes, linear)
TCP: Hash tables configured (established 8192 bind 8192)
UDP hash table entries: 512 (order: 2, 16384 bytes, linear)
UDP-Lite hash table entries: 512 (order: 2, 16384 bytes, linear)
NET: Registered PF_UNIX/PF_LOCAL protocol family
RPC: Registered named UNIX socket transport module.
RPC: Registered udp transport module.
RPC: Registered tcp transport module.
RPC: Registered tcp-with-tls transport module.
RPC: Registered tcp NFSv4.1 backchannel transport module.
armv7-pmu f8891000.pmu: hw perfevents: no interrupt-affinity property, guessing.
hw perfevents: enabled with armv7_cortex_a9 PMU driver, 7 counters available
workingset: timestamp_bits=30 max_order=18 bucket_order=0
jffs2: version 2.2. (NAND) (SUMMARY)  © 2001-2006 Red Hat, Inc.
fuse: init (API version 7.39)
bounce: pool size: 64 pages
io scheduler mq-deadline registered
io scheduler kyber registered
io scheduler bfq registered
zynq-pinctrl 700.pinctrl: zynq pinctrl initialized
dma-pl330 f8003000.dma-controller: Loaded driver for PL330 DMAC-241330
dma-pl330 f8003000.dma-controller:      DBUFF-128x8bytes Num_Chans-8 Num_Peri-4 Num_Events-16
brd: module loaded
loop: module loaded
macb e000b000.ethernet eth0: Cadence GEM rev 0x00020118 at 0xe000b000 irq 39 (00:0a:35:00:1e:53)
i2c_dev: i2c /dev entries driver
cdns-wdt f8005000.watchdog: Xilinx Watchdog Timer with timeout 10s
EDAC MC: ECC not enabled
Xilinx Zynq CpuIdle Driver started
sdhci: Secure Digital Host Controller Interface driver
sdhci: Copyright(c) Pierre Ossman
sdhci-pltfm: SDHCI platform and OF driver helper
ledtrig-cpu: registered to indicate activity on CPUs
clocksource: ttc_clocksource: mask: 0xffff max_cycles: 0xffff, max_idle_ns: 537538477 ns
timer #0 at (ptrval), irq=42
fpga_manager fpga0: Xilinx Zynq FPGA Manager registered
NET: Registered PF_INET6 protocol family
mmc0: SDHCI controller on e0100000.mmc [e0100000.mmc] using ADMA
Segment Routing with IPv6
In-situ OAM (IOAM) with IPv6
sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
NET: Registered PF_PACKET protocol family
Registering SWP/SWPB emulation handler
mmc0: new high speed SDHC card at address 59b4
mmcblk0: mmc0:59b4 SD32G 29.1 GiB
 mmcblk0: p1 p2
of-fpga-region fpga-region: FPGA Region probed
of_cfs_init
of_cfs_init: OK
macb e000b000.ethernet eth0: PHY [e000b000.ethernet-ffffffff:0c] driver [Generic PHY] (irq=POLL)
macb e000b000.ethernet eth0: configuring for phy/rgmii-id link mode
macb e000b000.ethernet eth0: Link is Up - 1Gbps/Full - flow control off
IP-Config: Complete:
     device=eth0, hwaddr=00:0a:35:00:1e:53, ipaddr=10.9.8.9, mask=255.255.255.0, gw=10.9.8.254
     host=10.9.8.9, domain=, nis-domain=(none)
     bootserver=10.9.8.8, rootserver=10.9.8.8, rootpath=
clk: Disabling unused clocks
VFS: Mounted root (nfs filesystem) on device 0:14.
devtmpfs: mounted
Freeing unused kernel image (initmem) memory: 1024K
petalinux:~$

```

</details>

### konosubakonoakua · 2024-09-25T22:43:17Z

## Appendix
### connect to outside network (reach the Internet 😊)
First, follow this guide #95 
Second, boot petalinux via nfs, login
Third, Add new ip address to `eth0`
check current settings
```bash
ifconfig eth0
```
```bash
sudo ip address add 192.168.137.253/24 brd + dev eth0
sudo ip route del default
sudo ip route add default via 192.168.137.1 dev eth0
```
```bash
ping 8.8.8.8 # should succeed
```
Fourth, add nameserver
```bash
echo "nameserver 192.168.137.1" | sudo tee -a /etc/resolv.conf
```
```bash
curl www.baidu.com # should succeed
```
But, failed to download from github 🧐🧐🧐.
```bash
petalinux:~/Downloads$ wget https://github.com/junegunn/fzf/releases/download/v0.55.0/fzf-0.55.0-linux_armv7.tar.gz
--2018-03-09 13:12:57--  https://github.com/junegunn/fzf/releases/download/v0.55.0/fzf-0.55.0-linux_armv7.tar.gz
Resolving github.com... 20.205.243.166
Connecting to github.com|20.205.243.166|:443... connected.
ERROR: The certificate of 'github.com' is not trusted.
ERROR: The certificate of 'github.com' is not yet activated.
The certificate has not yet been activated
The certificate has expired
```
<details>
<summary> fzf running on petalinux </summary>

<img src="https://github.com/user-attachments/assets/5a021bb9-6164-4c44-b2d7-3e713ad2a305" width="50%" />

</details>



### konosubakonoakua · 2024-09-26T04:11:40Z

## references
- [csdn petalinux tutorial](https://blog.csdn.net/small_po_kid/article/details/120325786)
  - [petalinux-tutorial-from-CSDN.pdf](https://github.com/konosubakonoakua/archives/releases/download/petalinux-tutorials/petalinux-tutorial-from-CSDN.pdf)
- [從一開始的 Xilinx SoC 開發，PetaLinux 使用（一）](https://ys-hayashi.me/2021/08/xilinx-petalinux-01/)
- [alinx 7015b 使用 petalinux 定制 linux 系统](https://ax7015b-20231-v101.readthedocs.io/zh-cn/latest/7015B_S4_RSTdocument_CN/05_%E4%BD%BF%E7%94%A8Petalinux%E5%AE%9A%E5%88%B6Linux%E7%B3%BB%E7%BB%9F_CN.html#petalinuxlinux)
- [xilinx-wiki PetaLinux Yacto Tips](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842475/PetaLinux+Yocto+Tips)
- [PetaLinux Tools for Embedded Linux Development Design Hub (DH246) (need login)](https://docs.amd.com/v/u/en-US/dh0016-petalinux-tools-hub)
- [hellofpga](http://www.hellofpga.com/)
  - [hellofpga petalinux tf boot](http://www.hellofpga.com/index.php/2023/04/28/petalinux_tf_boot/)
- boot issues
  - [kernel/nfsroot](https://www.kernel.org/doc/html/v6.11/admin-guide/nfs/nfsroot.html)
  - [pxelinux issue](https://adaptivesupport.amd.com/s/question/0D52E00007G0swPSAR/how-do-i-configure-petalinux-to-use-nfs-as-the-root-filesystem?language=en_US)
  - [Setting a Static IP Address Using the Kernel Command Line
  ](https://linuxlink.timesys.com/docs/static_ip)
  - [set-static-ip-address](https://www.baeldung.com/linux/set-static-ip-address)

### konosubakonoakua · 2024-12-12T01:20:36Z

## Trouble shooting
### Petalinux SSH login always fails the first time
- [petalinux-ssh-login-always-fails-the-first-time](https://adaptivesupport.amd.com/s/question/0D52E00006hpWoKSAU/petalinux-ssh-login-always-fails-the-first-time?language=en_US)
- [dropbear-root-login-petalinux-20191](https://adaptivesupport.amd.com/s/question/0D52E00006hplv7SAA/dropbear-root-login-petalinux-20191?language=en_US)

### konosubakonoakua · 2024-12-12T07:22:58Z

## Qemu
- https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/862912682/Networking+in+QEMU

### konosubakonoakua · 2025-02-12T02:37:47Z

## SD Card Preparation
- [How+to+format+SD+card+for+SD+boot](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842385/How+to+format+SD+card+for+SD+boot)
- [[sd_format_fat32-ext4.sh](https://github.com/konosubakonoakua/imp-fpga-download/blob/main/sd_format_fat32-ext4.sh)](https://github.com/Xilinx/PYNQ/blob/master/sdbuild/scripts/create_partitions.sh)


### konosubakonoakua · 2025-02-19T02:54:24Z

## Tips
- https://www.fpgadeveloper.com/adding-a-script-to-the-root-file-system-in-petalinux/
- https://www.fpgadeveloper.com/how-to-modify-u-boot-environment-variables-in-petalinux/

### konosubakonoakua · 2025-02-26T15:24:47Z

## patch petalinux
- https://www.fpgadeveloper.com/how-to-patch-petalinux/

---
number: 50
title: "[linux][kernel][qemu] qemu linux kernel development environment setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/50
author: konosubakonoakua
createdAt: 2024-05-15T05:09:07Z
updatedAt: 2024-05-17T07:30:43Z
labels: []
---

# 50 [linux][kernel][qemu] qemu linux kernel development environment setup

## file share
### loop device
### 9p
### samba
- https://www.cnblogs.com/harrypotterjackson/p/17469404.html
### nfs
- https://ubuntu.com/server/docs/network-file-system-nfs
### virtiofsd
- https://github.com/ubports/ubports-pdk/blob/main/scripts/images.sh
- https://github.com/nathanchance/env/blob/main/python/scripts/cbl_vmm.py
- https://virtio-fs.gitlab.io/howto-qemu.html
https://github.com/nathanchance/env/blob/d24019088f898a46d21bbe279e438e2f4bb7a50a/python/scripts/cbl_vmm.py#L122
- https://gitlab.com/virtio-fs/virtio-fs-ci/-/blob/master/build-linux.sh?ref_type=heads

---

## Comments

### konosubakonoakua · 2024-05-15T05:23:30Z

## raspberry pi
### kernel compile
- https://www.raspberrypi.com/documentation/computers/linux_kernel.html
- https://embear.ch/blog/compiling-a-kernel-module
- https://nayab.xyz/book/embedded-linux-rpi3-000-intro#learning-path
### pi4
- https://gist.github.com/cGandom/23764ad5517c8ec1d7cd904b923ad863
### pi3
- https://interrupt.memfault.com/blog/emulating-raspberry-pi-in-qemu
- https://github.com/KhalilOuali/Raspbian-Bullseye-with-QEMU?tab=readme-ov-file

### busybox + kernel
<details>
<summary>raspi aarch64 busybox kernel compiling, qemu booting</summary>

```shell
# # https://www.raspberrypi.com/documentation/computers/linux_kernel.html

alias makej='make -j$(nproc)'
export ROOTFS=$HOME/prj/linux/busybox/busybox-1.36.1/_install
export KERNEL=kernel8

####################################
# 32-bit configs
export ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- IMAGE=zImage
# For Raspberry Pi 1, Zero and Zero W, and for Raspberry Pi Compute Module 1:
#export KERNEL=kernel && make -j$(nproc) bcmrpi_defconfig
# For Raspberry Pi 2, 3, 3+ and Zero 2 W, and Raspberry Pi Compute Modules 3 and 3+:
export KERNEL=kernel7 && make -j$(nproc) bcm2709_defconfig
# For Raspberry Pi 4 and 400, and Raspberry Pi Compute Module 4:
#export KERNEL=kernel7l && make -j$(nproc) bcm2711_defconfig
make -j$(nproc) menuconfig
make -j$(nproc) zImage modules dtbs
####################################


####################################
# 64-bit configs
export ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- IMAGE=Image
# For Raspberry Pi 3, 3+, 4, 400 and Zero 2 W, and Raspberry Pi Compute Modules 3, 3+ and 4:
export KERNEL=kernel8 && make -j$(nproc) bcm2711_defconfig
# For Raspberry Pi 5:
#export KERNEL=kernel_2712 && make -j$(nproc) bcm2712_defconfig
make -j$(nproc) menuconfig
make -j$(nproc) Image modules dtbs
####################################


####################################
# kernel modules install
make INSTALL_MOD_PATH=$ROOTFS modules_install
####################################

####################################
# busybox
rm -rf $ROOTFS/boot
mkdir -p $ROOTFS/boot/overlays
cp -f arch/$ARCH/boot/$IMAGE $ROOTFS/boot/$KERNEL.img
cp -f arch/$ARCH/boot/dts/overlays/*.dtb* arch/$ARCH/boot/dts/overlays/README $ROOTFS/boot/overlays/
ls $ROOTFS/boot/
####################################


####################################
# virtiofsd
export VIRTIOFS_QUEUE_SIZE=1024 QEMU_VIRTIOFSD_SOCKET=/tmp/myfs.sock QEMU_VIRTIOFSD_CHARDEV_ID=virtiofs_chr0
export QEMU_VIRTIOFSD_TAG=myfs
export VIRTIOFS_QEMU_PARAM=(
  -chardev socket,id=virtiofs_chr0,path=$QEMU_VIRTIOFSD_SOCKET
  -device vhost-user-fs-pci,queue-size=$VIRTIOFS_QUEUE_SIZE,chardev=virtiofs_chr0,tag=myfs,ats=off
)
virtiofsd --socket-path=$QEMU_VIRTIOFSD_SOCKET -o source=$ROOTFS -o cache=none &
####################################


####################################
# qemu
#
    # -drive format=raw,file=2023-05-03-raspios-bullseye-arm64.img,if=none,id=hd0,cache=writeback \
    # -device virtio-blk,drive=hd0,bootindex=0 \
    # -netdev user,id=mynet,hostfwd=tcp::2222-:22 \
    # -device virtio-net-pci,netdev=mynet \
qemu-system-aarch64 -machine virt -cpu cortex-a53 -smp 4 -m 1G \
  -monitor telnet:127.0.0.1:5555,server,nowait\
  -kernel "$ROOTFS/boot/$KERNEL.img" \
  -append "rootfstype=virtiofs root=myfs rw panic=0 console=ttyAMA0" \
  -chardev socket,id=virtiofs_chr0,path=$QEMU_VIRTIOFSD_SOCKET \
  -device vhost-user-fs-pci,queue-size=$VIRTIOFS_QUEUE_SIZE,chardev=virtiofs_chr0,tag=myfs
  -object memory-backend-memfd,id=mem,size=4G,share=on \
  -numa node,memdev=mem
####################################

qemu-system-aarch64 -machine virt -cpu cortex-a53 -smp 4 -m 1G \
  -monitor telnet:127.0.0.1:5555,server,nowait\
  -kernel kernel.bin\
  -append "rootfstype=virtiofs root=myfs rw panic=0 console=ttyAMA0" \
  -chardev socket,id=virtiofs_chr0,path=/tmp/myfs.sock \
  -device vhost-user-fs-pci,queue-size=1024,chardev=virtiofs_chr0,tag=myfs

qemu-system-aarch64 -machine raspi3b -cpu cortex-a53 -smp 4 -m 1G \
  -kernel arch/$ARCH/boot/$IMAGE \
  -dtb arch/arm64/boot/dts/broadcom/bcm2710-rpi-3-b.dtb \
  -initrd $ROOTFS/../rootfs.img.gz \
  -append 'root=/dev/ram rdinit=/sbin/init panic=0' \
  -serial stdio


# -initrd $ROOTFS/../rootfs.img.gz \
# -append 'root=/dev/ram rdinit=/sbin/init panic=0 console=ttyAMA0' \
#
# -monitor telnet:localhost:4322,server,nowait \
# -serial telnet:localhost:4321,server,nowait \
# &
# sleep 2
# telnet localhost 4321

export ROOTFS=/home/rfe1wx/prj/linux/busybox/busybox-1.36.1/_install/
export ARCH=arm64 IMAGE=Image
virtiofsd --tag=myvfs --socket-path=/tmp/myvfs.sock --shared-dir=$(pwd) --cache=auto &
qemu-system-aarch64 -machine virt -cpu cortex-a53 -smp 4 -m 1G \
  -kernel arch/$ARCH/boot/$IMAGE \
  -append 'rootfstype=virtiofs root=myvfs rw panic=0 console=ttyAMA0' \
  -chardev socket,id=myvfs0,path=/tmp/myvfs.sock \
  -device vhost-user-fs-pci,queue-size=1024,chardev=myvfs0,tag=myvfs \
  -object memory-backend-memfd,id=mem,size=1G,share=on \
  -numa node,memdev=mem \
  -serial mon:stdio \
  -nographic
# Ctrl+a, x to quit


# cloud-hypervisor \
#   --cpus boot=4 \
#   --memory size=512M \
#   --serial tty \
#   --console off \
#   --kernel kernel.bin \
#   --disk path=rootfs.ext4 \
#   --cmdline "keep_bootcon console=hvc0 reboot=k panic=1 pci=off root=/dev/vda rw"

````

</details>

## Orange Pi
- https://wp.dejvino.com/2020/07/orange-pi-zero-running-in-qemu/

## x86_64
<details>
<summary>tutorials</summary>

- [build-a-kernel-initramfs-and-busybox-to-create-your-own-micro-linux](https://cylab.be/blog/320/build-a-kernel-initramfs-and-busybox-to-create-your-own-micro-linux?accept-cookies=1)
</details>

<details>
<summary>busybox + x86</summary>

### busybox
#### command
```shell
find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../rootfs

```
#### _install/init
```shell
#!/bin/sh

mount -t devtmpfs devtmpfs /dev
mount -t proc none /proc
mount -t sysfs none /sys

cat <<EOF

##################################################
#        Welcome to Micro Linux!
#  Boot took $(cut -d' ' -f1 /proc/uptime) seconds
##################################################

EOF

modprobe virtiofs && \
  mkdir /mnt && \
  mount -t virtiofs myfs /mnt && \
  echo && \
  echo "##################################################" && \
  echo "virtiofs mounted at /mnt" && \
  echo "##################################################" && \
  echo

mdev -s

exec /bin/sh
```

</details>


### konosubakonoakua · 2024-05-15T06:20:49Z

## building
### kernel
- https://docs.kernel.org/kbuild/modules.html
- https://mgalgs.io/2015/05/16/how-to-build-a-custom-linux-kernel-for-qemu-2015-edition.html
### initramfs
- https://wiki.gentoo.org/wiki/Custom_Initramfs
### rootfs
- https://cloud-images.ubuntu.com/minimal/releases/bionic/release/
### busybox
- https://www.vinnie.work/blog/2020-12-27-a-simple-busybox-system
- https://cylab.be/blog/332/add-modules-to-your-micro-linux
- https://re-ws.pl/2020/11/busybox-based-linux-distro-from-scratch/


### konosubakonoakua · 2024-05-16T08:52:26Z

## VM
### kvm
<details>
<summary>virt-manager</summary>

```shell
# Install Necessary KVM-Related Packages on a Debian-Based System:
# You can use a package manager like APT on Debian-based systems.
sudo apt-get install qemu-kvm libvirt-bin virt-manager

# Create a Virtual Disk Image:
# Here, we create a 20GB QCOW2 image called "myvm.img."
qemu-img create -f qcow2 myvm.img 20G

# Create a Virtual Machine:
# Use virt-install to create a KVM virtual machine. Adjust the parameters as needed for your use case.
# In this example, we create a VM named "myvm" with 2GB of RAM and 2 vCPUs, and we use an Ubuntu 18.04 ISO for installation.
virt-install \\
  --name myvm \\
  --ram 2048 \\
  --vcpus 2 \\
  --disk path=myvm.img,size=20 \\
  --os-type linux \\
  --os-variant ubuntu18.04 \\
  --network bridge=br0 \\
  --graphics vnc \\
  --console pty,target_type=serial \\
  --location '<http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/>'

# Start the Virtual Machine:
# Once the VM is created, you can start it using the following command:
virsh start myvm

# Connect to the VM's Console:
# To connect to the VM's console and interact with it, you can use:
virsh console myvm
```

</details>

___

### [cloudhypervisor](https://www.cloudhypervisor.org/docs/prologue/quick-start/)

<details>
<summary>cloud-hypervisor</summary>

```shell
$ ./cloud-hypervisor \
	--kernel ./linux-cloud-hypervisor/arch/arm64/boot/Image \
	--console off \
	--serial tty \
	--disk path=focal-server-cloudimg-arm64.raw \
	--cmdline "console=ttyAMA0 root=/dev/vda1 rw" \
	--cpus boot=4 \
	--memory size=1024M \
	--net "tap=,mac=,ip=,mask="
```

</details>

---

### qemu
#### qemu tutorial
- https://blog.jitendrapatro.me/emulating-aarch64arm64-with-qemu-part-1/
- https://doc.opensuse.org/documentation/leap/virtualization/html/book-virtualization/cha-qemu-running.html
- https://www.qemu.org/2020/11/04/osv-virtio-fs/

---


### konosubakonoakua · 2024-05-16T10:51:48Z

## others
### vscode
- https://zhuanlan.zhihu.com/p/510289859
- https://zhuanlan.zhihu.com/p/594235031
### misc
- https://0xax.gitbooks.io/linux-insides/content/
- https://krinkinmu.github.io/2020/12/26/position-independent-executable.html
- https://github.com/openeuler-mirror/stratovirt
- https://vmsplice.net/~stefan/virtio-fs_%20A%20Shared%20File%20System%20for%20Virtual%20Machines.pdf
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/kernel_administration_guide/chap-documentation-kernel_administration_guide-working_with_kernel_modules#doc-wrapper

### konosubakonoakua · 2024-05-16T17:51:24Z

## bug,issue,patch
currently, succeed to compile and run virtiofs for kernel, but virtio-fs cannot find tag given by the cmdline arg.

<details>
<summary>log</summary>

```shell
[2024-05-16T16:58:54Z INFO  virtiofsd] Waiting for vhost-user socket connection...
[2024-05-16T16:58:54Z INFO  virtiofsd] Client connected, servicing requests
qemu-system-aarch64: -device vhost-user-fs-device,queue-size=1024,chardev=myvfs0,tag=myvfs: warning: vhost-user b
ackend supports VHOST_USER_PROTOCOL_F_CONFIG but QEMU does not.
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd034]
[    0.000000] Linux version 6.1.77-v8+ (rfe1wx@WX8-V-0001N) (aarch64-none-linux-gnu-gcc (Arm GNU Toolchain 13.2.
rel1 (Build arm-13.7)) 13.2.1 20231009, GNU ld (Arm GNU Toolchain 13.2.rel1 (Build arm-13.7)) 2.41.0.20231009) #6
 SMP PREEMPT Fri May 17 00:56:55 CST 2024
[    0.000000] random: crng init done
[    0.000000] Machine model: linux,dummy-virt
[    0.000000] efi: UEFI not found.
[    0.000000] Zone ranges:
[    0.000000]   DMA      [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000]   DMA32    empty
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000040000000-0x000000007fffffff]
[    0.000000] cma: Reserved 8 MiB at 0x000000007e400000
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: PSCIv1.1 detected in firmware.
[    0.000000] psci: Using standard PSCI v0.2 function IDs
[    0.000000] psci: Trusted OS migration not required
[    0.000000] psci: SMC Calling Convention v1.0
[    0.000000] percpu: Embedded 29 pages/cpu s79144 r8192 d31448 u118784
[    0.000000] Detected VIPT I-cache on CPU0
[    0.000000] CPU features: kernel page table isolation forced ON by KASLR
[    0.000000] CPU features: detected: Kernel page table isolation (KPTI)
[    0.000000] CPU features: detected: ARM erratum 845719
[    0.000000] alternatives: applying boot alternatives
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 258048
[    0.000000] Kernel command line: rootfstype=virtiofs root=myfs rw panic=0 console=ttyAMA0
[    0.000000] Dentry cache hash table entries: 131072 (order: 8, 1048576 bytes, linear)
[    0.000000] Inode-cache hash table entries: 65536 (order: 7, 524288 bytes, linear)
[    0.000000] mem auto-init: stack:all(zero), heap alloc:off, heap free:off
[    0.000000] Memory: 996844K/1048576K available (11520K kernel code, 1788K rwdata, 3684K rodata, 3712K init, 10
77K bss, 43540K reserved, 8192K cma-reserved)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] ftrace: allocating 37325 entries in 146 pages
[    0.000000] ftrace: allocated 146 pages with 3 groups
[    0.000000] trace event string verifier disabled
[    0.000000] rcu: Preemptible hierarchical RCU implementation.
[    0.000000] rcu:     RCU event tracing is enabled.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=256 to nr_cpu_ids=4.
[    0.000000]  Trampoline variant of Tasks RCU enabled.
[    0.000000]  Rude variant of Tasks RCU enabled.
[    0.000000]  Tracing variant of Tasks RCU enabled.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
[    0.000000] NR_IRQS: 64, nr_irqs: 64, preallocated irqs: 0
[    0.000000] Root IRQ handler: gic_handle_irq
[    0.000000] GICv2m: range[mem 0x08020000-0x08020fff], SPI[80:143]
[    0.000000] rcu: srcu_init: Setting srcu_struct sizes based on contention.
[    0.000000] arch_timer: cp15 timer(s) running at 62.50MHz (virt).
[    0.000000] clocksource: arch_sys_counter: mask: 0x1ffffffffffffff max_cycles: 0x1cd42e208c, max_idle_ns: 8815
90405314 ns
[    0.000064] sched_clock: 57 bits at 63MHz, resolution 16ns, wraps every 4398046511096ns
[    0.006445] Console: colour dummy device 80x25
[    0.007285] Calibrating delay loop (skipped), value calculated using timer frequency.. 125.00 BogoMIPS (lpj=25
0000)
[    0.007432] pid_max: default: 32768 minimum: 301
[    0.008020] LSM: Security Framework initializing
[    0.010918] Mount-cache hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.010954] Mountpoint-cache hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.022082] cgroup: Disabling memory control group subsystem
[    0.034556] cacheinfo: Unable to detect cache hierarchy for CPU 0
[    0.039516] cblist_init_generic: Setting adjustable number of callback queues.
[    0.039533] cblist_init_generic: Setting shift to 2 and lim to 1.
[    0.039754] cblist_init_generic: Setting adjustable number of callback queues.
[    0.039761] cblist_init_generic: Setting shift to 2 and lim to 1.
[    0.040200] cblist_init_generic: Setting adjustable number of callback queues.
[    0.040208] cblist_init_generic: Setting shift to 2 and lim to 1.
[    0.041103] rcu: Hierarchical SRCU implementation.
[    0.041132] rcu:     Max phase no-delay instances is 1000.
[    0.044119] EFI services will not be available.
[    0.045078] smp: Bringing up secondary CPUs ...
[    0.047405] Detected VIPT I-cache on CPU1
[    0.047947] cacheinfo: Unable to detect cache hierarchy for CPU 1
[    0.048207] CPU1: Booted secondary processor 0x0000000001 [0x410fd034]
[    0.051855] Detected VIPT I-cache on CPU2
[    0.052000] cacheinfo: Unable to detect cache hierarchy for CPU 2
[    0.052314] CPU2: Booted secondary processor 0x0000000002 [0x410fd034]
[    0.053550] Detected VIPT I-cache on CPU3
[    0.053705] cacheinfo: Unable to detect cache hierarchy for CPU 3
[    0.053831] CPU3: Booted secondary processor 0x0000000003 [0x410fd034]
[    0.054441] smp: Brought up 1 node, 4 CPUs
[    0.054468] SMP: Total of 4 processors activated.
[    0.054532] CPU features: detected: 32-bit EL0 Support
[    0.054544] CPU features: detected: 32-bit EL1 Support
[    0.054584] CPU features: detected: CRC32 instructions
[    0.059047] CPU: All CPU(s) started at EL1
[    0.059172] alternatives: applying system-wide alternatives
[    0.077746] devtmpfs: initialized
[    0.084331] Enabled cp15_barrier support
[    0.084543] Enabled setend support
[    0.085308] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.085476] futex hash table entries: 1024 (order: 4, 65536 bytes, linear)
[    0.088006] pinctrl core: initialized pinctrl subsystem
[    0.095251] DMI not present or invalid.
[    0.102137] NET: Registered PF_NETLINK/PF_ROUTE protocol family
[    0.123139] DMA: preallocated 128 KiB GFP_KERNEL pool for atomic allocations
[    0.131888] DMA: preallocated 128 KiB GFP_KERNEL|GFP_DMA pool for atomic allocations
[    0.132158] DMA: preallocated 128 KiB GFP_KERNEL|GFP_DMA32 pool for atomic allocations
[    0.132483] audit: initializing netlink subsys (disabled)
[    0.135135] audit: type=2000 audit(0.104:1): state=initialized audit_enabled=0 res=1
[    0.141066] thermal_sys: Registered thermal governor 'step_wise'
[    0.141414] cpuidle: using governor menu
[    0.142326] hw-breakpoint: found 6 breakpoint and 4 watchpoint registers.
[    0.144656] ASID allocator initialised with 32768 entries
[    0.147476] Serial: AMBA PL011 UART driver
[    0.166773] 9000000.pl011: ttyAMA0 at MMIO 0x9000000 (irq = 13, base_baud = 0) is a PL011 rev1
[    0.183421] printk: console [ttyAMA0] enabled
[    0.191239] KASLR enabled
[    0.235550] iommu: Default domain type: Translated 
[    0.235762] iommu: DMA domain TLB invalidation policy: strict mode 
[    0.237203] SCSI subsystem initialized
[    0.238261] usbcore: registered new interface driver usbfs
[    0.238612] usbcore: registered new interface driver hub
[    0.238849] usbcore: registered new device driver usb
[    0.239748] pps_core: LinuxPPS API ver. 1 registered
[    0.239848] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.240096] PTP clock support registered
[    0.250926] vgaarb: loaded
[    0.253956] clocksource: Switched to clocksource arch_sys_counter
[    0.294826] VFS: Disk quotas dquot_6.6.0
[    0.295380] VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
[    0.296482] FS-Cache: Loaded
[    0.297654] CacheFiles: Loaded
[    0.324139] NET: Registered PF_INET protocol family
[    0.325872] IP idents hash table entries: 16384 (order: 5, 131072 bytes, linear)
[    0.332629] tcp_listen_portaddr_hash hash table entries: 512 (order: 1, 8192 bytes, linear)
[    0.332905] Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.333184] TCP established hash table entries: 8192 (order: 4, 65536 bytes, linear)
[    0.333487] TCP bind hash table entries: 8192 (order: 6, 262144 bytes, linear)
[    0.333848] TCP: Hash tables configured (established 8192 bind 8192)
[    0.335793] MPTCP token hash table entries: 1024 (order: 2, 24576 bytes, linear)
[    0.336351] UDP hash table entries: 512 (order: 2, 16384 bytes, linear)
[    0.336807] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes, linear)
[    0.338590] NET: Registered PF_UNIX/PF_LOCAL protocol family
[    0.339253] PCI: CLS 0 bytes, default 64
[    0.345166] hw perfevents: enabled with armv8_pmuv3 PMU driver, 7 counters available
[    0.346258] kvm [1]: HYP mode not available
[    0.349519] Initialise system trusted keyrings
[    0.355126] workingset: timestamp_bits=46 max_order=18 bucket_order=0
[    0.362733] zbud: loaded
[    0.366603] fuse: init (API version 7.37)
[    0.371890] Key type asymmetric registered
[    0.372099] Asymmetric key parser 'x509' registered
[    0.374508] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 247)
[    0.375955] io scheduler mq-deadline registered
[    0.376200] io scheduler kyber registered
[    0.392951] vc-mem: phys_addr:0x00000000 mem_base=0x00000000 mem_size:0x00000000(0 MiB)
[    0.394612] cacheinfo: Unable to detect cache hierarchy for CPU 0
[    0.433056] brd: module loaded
[    0.458285] loop: module loaded
[    0.461613] Loading iSCSI transport class v2.0-870.
[    0.479555] usbcore: registered new device driver r8152-cfgselector
[    0.480203] usbcore: registered new interface driver r8152
[    0.480620] usbcore: registered new interface driver lan78xx
[    0.480939] usbcore: registered new interface driver smsc95xx
[    0.482189] dwc_otg: version 3.00a 10-AUG-2012 (platform bus)
[    0.484459] usbcore: registered new interface driver uas
[    0.484956] usbcore: registered new interface driver usb-storage
[    0.486785] mousedev: PS/2 mouse device common for all mice
[    0.489593] sdhci: Secure Digital Host Controller Interface driver
[    0.492257] sdhci: Copyright(c) Pierre Ossman
[    0.492837] sdhci-pltfm: SDHCI platform and OF driver helper
[    0.495263] ledtrig-cpu: registered to indicate activity on CPUs
[    0.497058] hid: raw HID events driver (C) Jiri Kosina
[    0.497812] usbcore: registered new interface driver usbhid
[    0.498205] usbhid: USB HID core driver
[    0.499864] NET: Registered PF_PACKET protocol family
[    0.500499] Key type dns_resolver registered
[    0.506127] registered taskstats version 1
[    0.506673] Loading compiled-in X.509 certificates
[    0.511253] Key type .fscrypt registered
[    0.511430] Key type fscrypt-provisioning registered
[    0.527709] of_cfs_init
[    0.529552] of_cfs_init: OK
[    0.543193] uart-pl011 9000000.pl011: no DMA platform data
[    0.547956] virtio-fs: tag <myfs> not found
[    0.548910] virtio-fs: tag </dev/root> not found
[    0.549276] virtio-fs: tag </dev/root> not found
[    0.549511] List of all partitions:
[    0.550202] 0100            4096 ram0 
[    0.550288]  (driver?)
[    0.550633] 0101            4096 ram1 
[    0.550647]  (driver?)
[    0.550953] 0102            4096 ram2 
[    0.550964]  (driver?)
[    0.551228] 0103            4096 ram3 
[    0.551238]  (driver?)
[    0.552880] 0104            4096 ram4 
[    0.552894]  (driver?)
[    0.555123] 0105            4096 ram5 
[    0.555187]  (driver?)
[    0.555736] 0106            4096 ram6 
[    0.555781]  (driver?)
[    0.555914] 0107            4096 ram7 
[    0.555924]  (driver?)
[    0.556232] 0108            4096 ram8 
[    0.556241]  (driver?)
[    0.556548] 0109            4096 ram9 
[    0.556558]  (driver?)
[    0.556850] 010a            4096 ram10 
[    0.556861]  (driver?)
[    0.557149] 010b            4096 ram11 
[    0.557158]  (driver?)
[    0.557435] 010c            4096 ram12 
[    0.557445]  (driver?)
[    0.557771] 010d            4096 ram13 
[    0.557783]  (driver?)
[    0.558090] 010e            4096 ram14 
[    0.558100]  (driver?)
[    0.558389] 010f            4096 ram15 
[    0.558400]  (driver?)
[    0.558807] No filesystem could mount root, tried: 
[    0.558831]  virtiofs
[    0.559076] 
[    0.559463] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    0.561078] CPU: 2 PID: 1 Comm: swapper/0 Not tainted 6.1.77-v8+ #6
[    0.561373] Hardware name: linux,dummy-virt (DT)
[    0.561766] Call trace:
[    0.562125]  dump_backtrace.part.0+0xe8/0xf4
[    0.562534]  show_stack+0x20/0x30
[    0.562770]  dump_stack_lvl+0x64/0x80
[    0.562991]  dump_stack+0x18/0x34
[    0.563152]  panic+0x190/0x354
[    0.563318]  mount_block_root+0x228/0x248
[    0.563470]  mount_root+0x188/0x1a8
[    0.563893]  prepare_namespace+0x134/0x174
[    0.564280]  kernel_init_freeable+0x278/0x2a4
[    0.569294]  kernel_init+0x2c/0x140
[    0.569523]  ret_from_fork+0x10/0x20
[    0.570093] SMP: stopping secondary CPUs
[    0.570716] Kernel Offset: 0x1f77c00000 from 0xffffffc008000000
[    0.570996] PHYS_OFFSET: 0x40000000
[    0.571181] CPU features: 0x00000,00900080,0000421b
[    0.572134] Memory Limit: none
[    0.572638] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```

</details>

<details>

<summary>issues & patchs</summary>

- https://github.com/cloud-hypervisor/cloud-hypervisor/discussions/2719
- https://lore.kernel.org/linux-fsdevel/20231005203030.223489-1-vgoyal@redhat.com/
- https://gitlab.com/virtio-fs/virtiofsd/-/issues/128
- https://github.com/systemd/systemd-stable/pull/366/files
- https://github.com/systemd/mkosi/issues/2371
</details>


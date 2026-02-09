---
number: 152
title: "[fpga] petalinux issues"
state: open
url: https://github.com/konosubakonoakua/blog/issues/152
author: konosubakonoakua
createdAt: 2025-02-26T09:05:07Z
updatedAt: 2025-10-17T13:33:12Z
labels: []
---

# 152 [fpga] petalinux issues

## [Product Update Release Notes and Known Issues # Knowledge](https://adaptivesupport.amd.com/s/global-search/Product%20Update%20Release%20Notes%20and%20Known%20Issues?language=en_US)
- [000037095 - PetaLinux 2024.2 - Product Update Release Notes and Known Issues](https://adaptivesupport.amd.com/s/article/000037095?language=en_US)
- [76526 - PetaLinux 2021.1 - Product Update Release Notes and Known Issues](https://adaptivesupport.amd.com/s/article/76526?language=en_US)
- [72293 - PetaLinux 2019.1 - Product Update Release Notes and Known Issues](https://adaptivesupport.amd.com/s/article/72293?language=en_US)



---

## Comments

### konosubakonoakua · 2025-02-26T09:32:26Z

## `VFS: Cannot open root device "mmcblk1p2" or unknown-block(179,26)`
- wrong fs type
  > selected `INITRAMFS` but have no rootfs file found.
  > use another type, e.g. `INITRD`, or copy the rootfs file to SD.
  Or
  > device tree set wrong bootargs, check it at `components/plnx_workspace/device-tree/device-tree/system-conf.dtsi`
- sdcard level-shifter/write protect issue (fixed by dt)
  - [sd1-sd-20-partition-table-detected-by-uboot-but-not-by-linux-kernel](https://adaptivesupport.amd.com/s/question/0D52E00007AQD4hSAH/sd1-sd-20-partition-table-detected-by-uboot-but-not-by-linux-kernel)
  ```c
  &sdhci1 {
      no-1-8-v; // 1.8V to 3.3V level shifter (i.e. using LVCMOS18, but sd connector is driven by 3.3V)
      disable-wp;
      disable-cd;
  };
  ```


### konosubakonoakua · 2025-02-26T09:33:53Z

## petalinux 2019.1 fpga manager causes build error
- [petalinux-20191-fpga-manager-causes-build-error](https://adaptivesupport.amd.com/s/question/0D52E00006lLgw6SAC/petalinux-20191-fpga-manager-causes-build-error?language=en_US)
  
  > do not check `fpga manager` in `petalinux-config`

### konosubakonoakua · 2025-02-26T09:56:50Z

## petalinux 2019.1 device-tree (system-confi.dtsi) does not update automatically after re-config
Delete the device-tree at `components/plnx_workspace/device-tree/device-tree`, then `petalinux-build`

### konosubakonoakua · 2025-02-26T10:26:56Z

## messed up building
try 
```bash
petalinux-build -x mrproper
```

### konosubakonoakua · 2025-02-26T12:45:01Z

## petalinux booting hangs at `loop: module loaded`
- [zcu208-petalinux-20222-boot-hangs-at-loop-module-loaded](https://adaptivesupport.amd.com/s/question/0D54U00008gLStySAG/zcu208-petalinux-20222-boot-hangs-at-loop-module-loaded?language=en_US)
> Maybe because I didn't check `include bitstream` when exporting hardware.
> ![Image](../assets/152/1.png)
> If so, try to enable `petalinux-config --> DTG Settings --> Remove PL from devicetree`
> It's better to include bitstream?

### konosubakonoakua · 2025-03-14T08:22:03Z

## fpga manager sysfs write error: Connection timed out
- you may have a broken `.bin` file, check if you use the following command to generate bin files.
  ```bash
  bootgen -image $(BIF_FILE_PATH) -arch zynq -process_bitstream bin -w -o $(BIT2BIN_FILE)
  ```

### konosubakonoakua · 2025-03-19T00:09:42Z

## trenz petalinux issues
- https://wiki.trenz-electronic.de/display/PD/Petalinux+Troubleshoot

### konosubakonoakua · 2025-03-19T00:13:04Z

## petalinux can bootup but no uart output
- check pin assignment on zynq block design, it's easy to mis-click to another assignment.
- mio configurations here: `project-spec/hw-description/ps7_init.html`

### konosubakonoakua · 2025-03-19T00:44:54Z

## petalinux-create can't find `librdi_commonxillic.so`
try (I'm not sure, this error has no bad effect for me):
```bash
sudo echo "/opt/Xilinx/petalinux/tools/lib" > /etc/ld.so.conf.d/petalinux.so.conf
sudo ldconfig
```

### konosubakonoakua · 2025-03-19T01:17:37Z

## ERROR: You are not inside a PetaLinux project. Please specify a PetaLinux project!
- refer to [here](https://adaptivesupport.amd.com/s/question/0D52E00006hpM4VSAU/petalinux-version-control?language=en_US)
- you need to keep `.petalinux/metadata`

### konosubakonoakua · 2025-03-20T06:53:03Z

## pthread not found
- add ldflags to bb file
  ```bb
  LDFLAGS += "-pthread"
  ```

### konosubakonoakua · 2025-03-20T06:53:56Z

## need to import header from another location
- add CFLAGS to bb
  ```bb
  CFLAGS += " -I${TOPDIR}/../project-spec/meta-user/recipes-modules/dma-proxy/files/"
  ```

### konosubakonoakua · 2025-03-20T06:55:16Z

## petalinux app compile failed
- try to clean first
  ```bash
  petalinux-build -c app -x clean
  petalinux-build -c app
  ```

### konosubakonoakua · 2025-03-20T07:11:33Z

## petalinux yacto howto
- https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842475/PetaLinux+Yocto+Tips

### konosubakonoakua · 2025-03-20T09:14:23Z

## petalinux increase CMA size (default 16)
- [000034737 - PetaLinux 2022.1: How to increase the CMA size in the Linux kernel and its impacts](https://adaptivesupport.amd.com/s/article/000034737?language=en_US)
- in kernel menuconfig, serach `CMA_SIZE_MBYTES`

### konosubakonoakua · 2025-04-14T01:12:14Z

### build/misc/config/Kconfig:223: can't open file "build/misc/config/Kconfig.syshw"
- It's because you moved or renamed project, causing path inconsistency
- fixed by modifying  `<your-project-path>build/misc/config/Kconfig.syshw` in `build/misc/config/Kconfig.part`.

### konosubakonoakua · 2025-04-14T01:37:30Z

## petalinux 2021 sstate cache issues
### 2021.1 PetaLinux: Package do_fetch task executes even though PREMIRRORS set to downloads path for offline builds

<details>

https://adaptivesupport.amd.com/s/article/2021-1-PetaLinux?language=en_US
- Unset `CONFIG_PRE_MIRROR_URL` in `<plnx-proj-root>/project-spec/configs/config`
- $ `cat project-spec/meta-user/conf/petalinuxbsp.conf`
  ```bb
  #User Configuration
  
  #OE_TERMINAL = "tmux"
  
  IMAGE_BOOT_FILES_zynqmp = "BOOT.BIN boot.scr Image ramdisk.cpio.gz.u-boot"
  
  # Workaround solution for offline builds.
  INHERIT += "own-mirrors"
  SOURCE_MIRROR_URL = "file:///home/wtsemb/plnx-workspace/2021.1/downloads"
  PREMIRRORS_append = ""
  SSTATE_MIRRORS = ""
  DL_DIR = "/home/wtsemb/plnx-workspace/2021.1/plnx-downloads"
  ```
- delete `build/conf/site.conf`
  ```bash
  $ petalinux-build -x mrproper
  $ petalinux-config --silentconfig
  $ rm -rf build/conf/site.conf
  $ petalinux-build
  ```
</details>

### 2021.2 and 2022.x Yocto/PetaLinux: Build throws errors when PREMIRRORS is set to downloads path for offline builds
https://adaptivesupport.amd.com/s/article/000033124?language=en_US

### my solution

<details>

- mkdir:  (only usefule when download from web??? IDK!)
  ```bash
  mkdir -p /impcas/cache/petalinux/sstate/downloads
  mkdir -p /impcas/cache/petalinux/sstate/aarch64
  mkdir -p /impcas/cache/petalinux/sstate/arm
  ```
- in `petalinuxbsp.conf`: (only usefule when download from web??? IDK!)
  ```makefile
  DL_DIR = "/impcas/cache/petalinux/sstate/downloads"
  SSTATE_DIR = "/impcas/cache/petalinux/2021.1/sstate/aarch64"
  ```
- in`pealinuxbsp.conf` (not tried yet)
  ```makefile
  PREMIRRORS_prepend = " \
  git://.*/.* file:///mnt/petalinux/sstate/2021.1/downloads \n \
  gitsm://.*/.* file:///mnt/petalinux/sstate/2021.1/downloads \n \
  ftp://.*/.* file:///mnt/petalinux/sstate/2021.1/downloads \n \
  http://.*/.* file:///mnt/petalinux/sstate/2021.1/downloads \n \
  https://.*/.* file:///mnt/petalinux/sstate/2021.1/downloads \n"
  ```
- in menuconfig, DON NOT set `BB_NO_NETWORK`
  ```makefile
  #
  # Add pre-mirror url 
  #
  CONFIG_PRE_MIRROR_URL="file:///mnt/petalinux/sstate/2021.1/downloads"
  
  #
  # Local sstate feeds settings
  #
  CONFIG_YOCTO_LOCAL_SSTATE_FEEDS_URL="/mnt/petalinux/sstate/2021.1/aarch64"
  # CONFIG_YOCTO_NETWORK_SSTATE_FEEDS is not set
  # CONFIG_YOCTO_BB_NO_NETWORK is not set
  # CONFIG_YOCTO_BUILDTOOLS_EXTENDED is not set
  ```
- mount cifs to `/mnt/petalinux/sstate`
  ```bash
  sudo mount -t cifs //rf-pc-shiro.local/D/tools/petalinux/sstate /mnt/petalinux/sstate -o username=RF-PC-SHIRO
  ```
### refs
- [PetaLinuxYoctoTips-How to reduce build time using SSTATECACHE](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842475/PetaLinux+Yocto+Tips#PetaLinuxYoctoTips-HowtoreducebuildtimeusingSSTATECACHE)
- [ug1144-petalinux-tools-reference-guide/Shared-State-Cache](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Shared-State-Cache)
</details>



### konosubakonoakua · 2025-04-22T09:08:47Z

## how to not build rootfs in case user has their own
- https://adaptivesupport.amd.com/s/question/0D52E00006hpjHlSAI/petalinux-without-rootfs?language=en_US
- remove image fs types
  ```bash
  tee -a project-spec/meta-user/conf/petalinuxbsp.conf << 'EOF'
  IMAGE_FSTYPES_remove = " tar.gz cpio cpio.gz.u-boot"
  EOF
  ```
- disable ramfs in kernel
  ```txt
  General setup  --->   [ ] Initial RAM filesystem and RAM disk (initramfs/initrd) support 
  ```

### konosubakonoakua · 2025-05-09T07:02:37Z

## `do_assembel_fitimage` failed, `extend_recipe_sysroot(d)`
- sysroot softlink damaged
```bash
petalinux-build -x distclean
rm -rf build/tmp
petalinux-build --force
```

### konosubakonoakua · 2025-05-14T03:09:14Z

## `Error: There's no '/dev' on rootfs`
- https://adaptivesupport.amd.com/s/question/0D52E00006hprw1SAA/error-theres-no-dev-on-rootfs
  - set `petalinux-config -> Image packaging configuration -> ( petalinux-image-minimal ) INITRAMFS/INITRD`


### konosubakonoakua · 2025-10-17T13:18:27Z

## device tree not updated
Copy `system-user.dtsi` to `petalinux_project\components\plnx_workspace\device-tree\`
```bash
petalinux-build -c device-tree -x cleansstate
petalinux-build -c device-dree
```

### konosubakonoakua · 2025-10-17T13:23:21Z


## `Unknown command booti` when booting petalinux (u-boot stage)
<details>

- [zedboard-unknown-command-booti-petalinux](https://adaptivesupport.amd.com/s/question/0D52E00006hpjY0SAI/zedboard-unknown-command-booti-petalinux?language=en_US)
- https://adaptivesupport.amd.com/s/question/0D52E00006hphFiSAI/migrating-from-petalinux-20174-to-20181-extra-hoops-to-jump-through?language=zh_CN

  > After  run `petalinux-config -c u-boot`
  > `vim project-spec/meta-user/recipes-bsp/u-boot/files/platform-top.h`
  > Add the following code to bottom.
  > Run `petalinux-build -c u-boot` or `petalinux-build` rebuild uboot or rebuild all project
  > Package images finally: `petalinux-package --boot --fsbl --u-boot --force`
  
  ```c
  /* Due to a bug where having u-boot load dtb from SD card causes the boot
   * command to default to using booti instead of bootm on Zynq, the defult build
   * fails to boot. This boot command override is a temporary workaround.
  */
   #ifdef CONFIG_BOOTCOMMAND
   #undef CONFIG_BOOTCOMMAND
   #define CONFIG_BOOTCOMMAND	"run uenvboot; run cp_kernel2ram && run cp_dtb2ram && bootm ${netstart} - ${dtbnetstart}"
   #endif
  ```
</details>

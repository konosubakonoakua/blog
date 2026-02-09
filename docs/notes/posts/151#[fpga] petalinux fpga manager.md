---
number: 151
title: "[fpga] petalinux fpga manager"
state: open
url: https://github.com/konosubakonoakua/blog/issues/151
author: konosubakonoakua
createdAt: 2025-02-26T08:29:48Z
updatedAt: 2025-10-13T02:40:17Z
labels: []
---

# 151 [fpga] petalinux fpga manager

- [Solution+ZynqMP+PL+Programming](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841847/Solution+ZynqMP+PL+Programming)
- [Solution+Zynq+PL+Programming+With+FPGA+Manager](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841645/Solution+Zynq+PL+Programming+With+FPGA+Manager)
- [U-Boot+FPGA+Driver](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/124682257/U-Boot+FPGA+Driver)
- [FPGA-Manager-Configuration-and-Usage-for-Zynq-7000-Devices-and-Zynq-UltraScale-MPSoC](https://docs.amd.com/r/2021.1-English/ug1144-petalinux-tools-reference-guide/FPGA-Manager-Configuration-and-Usage-for-Zynq-7000-Devices-and-Zynq-UltraScale-MPSoC)
- [programming-the-pl-at-runtime-with-petalinux-72a820](https://www.hackster.io/anujvaishnav20/programming-the-pl-at-runtime-with-petalinux-72a820)
- [基于FPGA Manager的Zynq PL程序写入方案](https://www.cnblogs.com/luego/p/11777104.html)


---

## Comments

### konosubakonoakua · 2025-02-26T13:23:38Z

## pl.dtbo overlay
- [kria-apps-docs/dtsi_dtbo_generation](https://xilinx.github.io/kria-apps-docs/creating_applications/2022.1/build/html/docs/dtsi_dtbo_generation.html)
- [meta-xilinx-tools README.fpgamanager.dtg.dfx.md](https://github.com/Xilinx/meta-xilinx-tools/blob/rel-v2023.1/docs/README.fpgamanager.dtg.dfx.md)
- [ug1144-petalinux-tools-reference-guide/DTG-Settings](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/DTG-Settings)

### konosubakonoakua · 2025-02-26T13:46:58Z

## scripts
- generate bit wrapper bin file
  ```bash
  source /tools/Xilinx/Vivado/2019.1/settings64.sh
  cd petalinux/images/linux
  
  tee design_1_wrapper.bit.bif << EOF
  all:
  {
    ../../../project/project.runs/impl_1/design_1_wrapper.bit
  }
  EOF

  echo "all: { ../../../project/project.runs/impl_1/design_1_wrapper.bit }" design_1_wrapper.bit.bif

  bootgen -image design_1_wrapper.bit.bif -arch zynq -o ./design_1_wrapper.bit.bin -w
  ```
- flash bin to fpga
  ```bash
  BITSTREAM_NAME=design_1_wrapper.bit.bin
  cp /root/shared/$BITSTREAM_NAME /lib/firmware
  echo 0 > /sys/class/fpga_manager/fpga0/flags
  echo $BITSTREAM_NAME > /sys/class/fpga_manager/fpga0/firmware
  ```

### konosubakonoakua · 2025-02-26T15:10:08Z

## issues
- cannot download fpga via system interface
  ```bash
  ❯ echo $BITSTREAM_NAME > /sys/class/fpga_manager/fpga0/firmware
  fpga_manager fpga0: reading flags 0
  fpga_manager fpga0: writing design_1_wrapper.bit.bin to Xilinx Zynq FPGA Manager
  fpga_manager fpga0: Error after writing image data to FPGA
  -bash: echo: write error: Connection timed out
  
  ```
  - [0001-fix-zynq-pr.patch](https://github.com/Xilinx/PYNQ/blob/image_v2.4/sdbuild/boot/meta-pynq/recipes-kernel/linux/linux-xlnx/0001-fix-zynq-pr.patch)
  - [zynqmp_fix.patch](https://github.com/Xilinx/PYNQ/blob/image_v2.3/sdbuild/boot/meta-pynq/recipes-kernel/linux/linux-xlnx/zynqmp_fix.patch)

### konosubakonoakua · 2025-02-26T15:59:48Z

## fpgautil
- https://github.com/Xilinx/meta-xilinx/blob/master/meta-xilinx-core/recipes-bsp/fpga-manager-script/files/fpgautil.c

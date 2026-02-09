---
number: 100
title: "[sw][fpga][xilinx][petalinux] customizing PYNQ"
state: open
url: https://github.com/konosubakonoakua/blog/issues/100
author: konosubakonoakua
createdAt: 2024-09-25T18:29:55Z
updatedAt: 2025-12-31T15:06:10Z
labels: ["FPGA", "ZYNQ", "Linux", "python"]
---

# 100 [sw][fpga][xilinx][petalinux] customizing PYNQ

## Repositories
- https://github.com/konosubakonoakua/PYNQ-IMPCAS-ZU9
- https://github.com/konosubakonoakua/PYNQ-ANTMINER-T9

## How-Tos

### How to setup env
```bash
source /tools/Xilinx/Vitis/2024.1/settings64.sh
source /tools/Petalinux/2024.1/settings.sh
```
### How to Umount iamges before building again
```bash
cd PYNQ/sdbuild/
make unmount  # Unmount without deleting
make delete   # Unmount and delete failed images
make clean    # Remove all build artifacts and staged images
```

### How to customize BSP

<details>

#### Kernel 
```bash
meta-user/recipes-kernel/linux/linux-xlnx/
```
#### Device-tree
```bash
meta-user/recipes-bsp/device-tree/files/system-user.dtsi
```
#### Boot files
```bash
meta-user/recipes-bsp/fsbl/
```

</details>

### How to create your packages

<details>

#### Package Structure

- **Makefile**: Defines `PACKAGE_BUILD_${PACKAGE_NAME}` targets
- **pre.sh**: Script run before chroot (file staging)
- **qemu.sh**: Script run inside chroot (installation)
- **post.sh**: Script run after chroot (cleanup)

```text
# Example package directory structure
packages/
└── my_board_package/
    ├── Makefile
    ├── pre.sh
    ├── qemu.sh
    └── post.sh
```
#### Referencing Custom Packages
Add custom packages to the spec file:
```makefile
STAGE4_PACKAGES_${BOARD} := pynq my_board_package
```

</details>

### How to use prebuilt artifacts
Download them here: https://www.pynq.io/boards.html
```bash
# For rebuilding all SD cards, both arm and aarch64 root filesystems
# may be required depending on boards being targetted.
cp pynq_rootfs.<arm|aarch64>.tar.gz <PYNQ repository>/sdbuild/prebuilt/pynq_rootfs.<arm|aarch64>.tar.gz
cp pynq-<version>.tar.gz            <PYNQ repository>/sdbuild/prebuilt/pynq_sdist.tar.gz
```

### How to disable FPGA Manager
In `<BORAD>.spec`:
```make
FPGA_MANAGER_${BOARD} := 1
```

### How to change hostname
- `/boot/boot.py`
  ```py
  # Example boot.py modifications accessible at /boot/boot.py
  import os
  
  # Set hostname
  os.system('hostnamectl set-hostname my-custom-board')
  ```
- `pynq_hostname.sh`

### How to understand XSAs in `petalinux_bsp/hardware_project/` and `base/`

Aspect | hardware_project/ XSA | base/ XSA
-- | -- | --
Purpose | BSP generation for boot files | Runtime overlay for PYNQ applications
Usage | Creates Petalinux BSP with `petalinux-config --get-hw-description` in `create_bsp.sh:35-36` | Loaded by PYNQ Overlay class using `RuntimeMetadataParser` in `embedded_device.py:268-275`
Build Stage | Stage 1-2: Creates boot environment | Stage 4: Installed as overlay package
Content	| Complete hardware platform (PS + PL) | PL design with PYNQ IP interfaces


```text
hardware_project/design.xsa  →  BSP Creation  →  Boot Files  
base/base.xsa               →  PYNQ Package  →  Runtime Overlay  
```

### How to add base overlay
```
Pynq-IMPCAS-ZU9/base/
├── base.bit
├── base.hwh
├── base.py
├── base.xsa
├── __init__.py
├── Makefile
└── notebooks
    └── overlay_axigpio.ipynb

```

- You must notice that there's a `__init__.py` in `base/`.
- `notebooks/*` will be copied to `/home/xilinx/jupyter_notebooks/base/`.
- There are 4 mandatory files: `base.bit`, `base.hwh`, `base.py` and `__init__.py`.

### How to create a new hardware design for pynq
- https://discuss.pynq.io/t/tutorial-creating-a-hardware-design-for-pynq/145

---

## Comments

### konosubakonoakua · 2025-12-29T06:03:17Z

## Issues

### system-user.dtsi
Put into `~/PYNQ/boards/impcas_zu9/petalinux_bsp/meta-user/recipes-bsp/device-tree/files/system-user.dtsi`

<details>

```bash
/include/ "system-conf.dtsi"
/ {

	chosen {
		bootargs = "earlycon console=ttyPS0,115200 root=/dev/mmcblk0p2 ro rootwait clk_ignore_unused uio_pdrv_genirq.of_id=generic-uio";
		stdout-path = "serial0:115200n8";
	};

    fclk0: fclk0 {
        status = "okay";
        compatible = "xlnx,fclk";
        clock-names = "fclk0";
        clocks = <&zynqmp_clk PL0_REF>;
    };

};

&sdhci1 {
    no-1-8-v; // 1.8V to 3.3V level shifter (i.e. using LVCMOS18, but sd connector is driven by 3.3V)
    disable-wp;
    disable-cd;
};

&gem3 {
    phy-handle = <&phyc>;
    phyc: phy@c {
        reg = <0xc>;
        ti,rx-internal-delay = <0x8>;
        ti,tx-internal-delay = <0xa>;
        ti,fifo-depth = <0x1>;
        ti,dp83867-rxctrl-strap-quirk;
    };
    /* Cleanup from RevA */
    // /delete-node/ phy@21;
};
```

</details>

### DHCP

### ModuleNotFoundError: No module named 'pynq.overlays.base'
There's no `base.py` in your board folder.
<details>

```py
import pynq
import pynq.lib
from pynq.lib.logictools import TraceAnalyzer


class BaseOverlay(pynq.Overlay):
    """Custom base overlay for your board"""

    def __init__(self, bitfile, **kwargs):
        super().__init__(bitfile, **kwargs)
        if self.is_loaded():
            # Configure GPIO
            self._configure_gpio()

            # Configure IOPs
            self._configure_iops()

            # Configure subsystems
            self._configure_subsystems()

    def _configure_gpio(self):
        # Map GPIO channels and set directions
        pass

    def _configure_iops(self):
        # Set up MicroBlaze processors
        pass

    def _configure_subsystems(self):
        # Initialize audio, video, trace analyzers
        pass
```

</details>

### Hostname
```bash
sudo pynq_hostname.sh <new_hostname>
sudo shutdown -r now
```


### konosubakonoakua · 2025-12-29T08:57:23Z

## References
### Makefile Targets Reference

Target | Description
-- | --
make | Build complete SD card images for all boards
make BOARDS=<board> | Build image for specific board
make BOARDDIR=<path> | Build from external board repository
make boot_files | Generate only boot files (BOOT.BIN, image.ub)
make bsp | Generate Petalinux BSP package
make sysroot | Generate SDK sysroot for cross-compilation
make clean | Remove all build artifacts
make unmount | Unmount any mounted images
make delete | Unmount and delete failed images

### Environment Variables

Variable | Purpose | Default
-- | -- | --
BOARDDIR | External board repository path | <PYNQ>/boards/
BOARDS | Space-separated list of boards to build | All boards in BOARDDIR
REBUILD_PYNQ_ROOTFS | Force rebuild of base rootfs | False
REBUILD_PYNQ_SDIST | Force rebuild of PYNQ source distribution | False
PYNQ_SDIST | Path to prebuilt PYNQ sdist tarball | prebuilt/pynq_sdist.tar.gz
PYNQ_ROOTFS | Path to prebuilt rootfs tarball | prebuilt/pynq_rootfs.<arch>.tar.gz
PYNQ_UBUNTU_REPO | Ubuntu package repository URL | https://ports.ubuntu.com


### official tutorial
- https://deepwiki.com/Xilinx/PYNQ/9.3-adding-board-support
- https://www.pynq.io/boards.html
### 3rd tutorials
- https://github.com/kangyuzhe666/zynq7010-pynq-2.5
- https://blog.csdn.net/BUBUBUBUBUBUBUB/article/details/139840329
- https://zhutmost.com/post/pynq-compile
- https://blog.csdn.net/qq_41873311/article/details/124284037
- https://github.com/WangHaoZhe/PYNQ-Tutorial
- https://www.dspsandbox.org/building-pynq-for-cora-z7-07z/
- https://discuss.pynq.io/t/build-pynq-2-4-sd-card-image-for-zedboard-v2018-3/2702
- https://gist.github.com/PeterOgden/9a5054a06408d2bd711d6de563281930
- http://www.hellofpga.com/index.php/2023/08/18/pynq_part1/
  - http://www.hellofpga.com/index.php/2023/08/19/pynq_vivado
  - http://www.hellofpga.com/index.php/2023/08/21/pynq_img

### konosubakonoakua · 2025-12-30T05:15:33Z

## Issues
### [ERROR] module 'plnx_vars' has no attribute 'CopyDir'
- It's a [petalinux tftpboot bug in 2024.1 version](https://github.com/Xilinx/PetaLinux/pull/1/commits/3439b44a64dfaf923dbc3de5896b3a1863897495)

  ```bash
  sudo rm -rf /tftpboot
  ```
### Kernel Panic - not syncing: VFS: Unable to mount root fs on unknown-block(179,10)
- Case 1: device-tree sd-card node problem
  - Add patch
- Case 2: damaged rootfs partition on sd-card
  - Reduce block size, e.g. `bs=1M`

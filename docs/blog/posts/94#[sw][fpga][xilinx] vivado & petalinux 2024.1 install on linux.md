---
number: 94
title: "[sw][fpga][xilinx] vivado & petalinux 2024.1 install on linux"
state: open
url: https://github.com/konosubakonoakua/blog/issues/94
author: konosubakonoakua
createdAt: 2024-09-23T06:59:22Z
updatedAt: 2024-09-25T22:46:19Z
labels: ["FPGA", "ZYNQ", "Linux"]
---

# 94 [sw][fpga][xilinx] vivado & petalinux 2024.1 install on linux

refer to petalinux 2019
- https://github.com/konosubakonoakua/blog/issues/97#issuecomment-2374149784

---

## Comments

### konosubakonoakua · 2024-09-23T22:42:31Z

## downloads
https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/2024-1.html

## deps
~refer to 2019.1~

## install
```bash
cd ~/Downloads
tar xvfz FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023.tar.gz
cd FPGAs_AdaptiveSoCs_Unified_2024.1_0522_2023
sudo ./xsetup
```
## create desktop files (if not have them)
<details>

```bash
mkdir -p ~/.local/share/applications
cd ~/.local/share/applications/

cat <<EOF > vivado2024.1.desktop
[Desktop Entry]
Type=Application
Name=Vivado 2024.1
Exec=bash -c "source /tools/Xilinx/Model_Composer/2024.1/settings64.sh && source /tools/Xilinx/Vitis/2024.1/settings64.sh && source /tools/Xilinx/Vitis_HLS/2024.1/settings64.sh && source /tools/Xilinx/Vivado/2024.1/settings64.sh && /tools/Xilinx/Vivado/2024.1/bin/vivado"
Icon=/tools/Xilinx/Vivado/2024.1/doc/images/vivado_logo.png
StartupNotify=true
Categories=Application;
EOF

cat <<EOF > vitis2024.1.desktop
[Desktop Entry]
Type=Application
Name=Vitis 2024.1
Exec=bash -c "source /tools/Xilinx/Model_Composer/2024.1/settings64.sh && source /tools/Xilinx/Vitis/2024.1/settings64.sh && source /tools/Xilinx/Vitis_HLS/2024.1/settings64.sh && source /tools/Xilinx/Vivado/2024.1/settings64.sh && /tools/Xilinx/Vitis/2024.1/bin/vitis"
Icon=/tools/Xilinx/Vitis/2024.1/doc/images/ide_icon.ico
StartupNotify=true
Cat
egories=Application;
EOF

cat <<EOF > DocNav2024.1.desktop
[Desktop Entry]
Type=Application
Name=DocNav 2024.1
Exec=bash -c "source /tools/Xilinx/Model_Composer/2024.1/settings64.sh && source /tools/Xilinx/Vitis/2024.1/settings64.sh && source /tools/Xilinx/Vitis_HLS/2024.1/settings64.sh && source /tools/Xilinx/Vivado/2024.1/settings64.sh && /tools/Xilinx/Vitis/2024.1/bin/vitis"
Icon=/tools/Xilinx/DocNav/resources/doc_nav_application_48.png
StartupNotify=true
Categories=Application;
EOF

cat <<EOF > PDM2024.1.desktop
[Desktop Entry]
Type=Application
Name=PDM 2024.1
Exec=bash -c "source /tools/Xilinx/Model_Composer/2024.1/settings64.sh && source /tools/Xilinx/Vitis/2024.1/settings64.sh && source /tools/Xilinx/Vitis_HLS/2024.1/settings64.sh && source /tools/Xilinx/Vivado/2024.1/settings64.sh && /tools/Xilinx/Vitis/2024.1/bin/vitis"
Icon=/tools/Xilinx/PDM/2024.1/doc/images/pdm_logo64.png
StartupNotify=true
Categories=Application;
EOF
```

<details>

### konosubakonoakua · 2024-09-23T22:43:48Z

## petalinux 2024.1
create /tools/petalinux if not exists
```bash
sudo mkdir -p /tools/petalinux
sudo chown rf:rf /tools/petalinux
```
copy from windows smb
```bash
cd /mnt/winshare
cp petalinux-v2024.1-05202009-installer.run ~/Downloads/
cd ~/Downloads
mkdir -p /tools/petalinux/sstate/2024.1
./petalinux-v2024.1-05202009-installer.run -y -d /tools/petalinux/2024.1
```
fix tftpboot bug https://github.com/konosubakonoakua/blog/issues/94#issuecomment-2369992894

copy sstate files
```bash
cd /mnt/winshare
cp sstate_aarch64_2024.1_05201002.tar.gz sstate_arm_2024.1_05201002.tar.gz cp downloads_2024.1_05201002.tar.gz ~/Downloads
cp downloads_2024.1_05201002.tar.gz ~/Downloads
```
extract sstate files
```bash
tar xvzf sstate_aarch64_2024.1_05201002.tar.gz -C /tools/petalinux/sstate/2024.1/ &
tar vxzf sstate_arm_2024.1_05201002.tar.gz -C /tools/petalinux/sstate/ &
tar xvzf downloads_2024.1_05201002.tar.gz -C /tools/petalinux/sstate/2024.1/ &
```
Config petalinux projects: https://github.com/konosubakonoakua/blog/issues/89

### konosubakonoakua · 2024-09-24T14:23:43Z

## trouble-shooting
### petalinux-build failed with tftpboot enabled
```bash
[ERROR] module 'plnx_vars' has no attribute 'CopyDir'
```
solution here: https://github.com/Xilinx/PetaLinux/pull/1/files
shortly,
```shell
cp -f /tools/petalinux/2024.1/scripts/petalinux-build /tools/petalinux/2024.1/scripts/petalinux-build.bak
sed -i "s/plnx_vars.CopyDir/plnx_utils.CopyDir/" /tools/petalinux/2024.1/scripts/petalinux-build
```


### konosubakonoakua · 2024-09-25T14:04:20Z

## References
- https://elinux.org/TFTP_Boot_and_NFS_Root_Filesystems

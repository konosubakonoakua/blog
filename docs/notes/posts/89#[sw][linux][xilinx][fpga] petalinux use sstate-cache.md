---
number: 89
title: "[sw][linux][xilinx][fpga] petalinux use sstate-cache"
state: open
url: https://github.com/konosubakonoakua/blog/issues/89
author: konosubakonoakua
createdAt: 2024-09-22T03:21:57Z
updatedAt: 2024-09-25T22:46:54Z
labels: ["FPGA", "ZYNQ", "Linux"]
---

# 89 [sw][linux][xilinx][fpga] petalinux use sstate-cache

## use [sstate-cache]
(https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/)
```bash
$ petalinux-config ---> Yocto Settings ---> Add pre-mirror url ---> file:///tools/petalinux/sstate/2024.1/downloads
$ petalinux-config ---> Yocto Settings ---> Local sstate feeds settings ---> local sstate feeds url ---> /tools/petalinux/sstate/2024.1/arm
```
### 2024.1
```txt
By default petalinux uses sstate-cache and download mirrors from petalinux.xilinx.com
This README is for the users who uses tool completely offline.

Setting sstate cache

1) Extract the downloaded tar contents of sstate-cache at any location
2) run petalinux-config
         -> Yocto Settings
             ->Local sstate feeds settings
                  ->local sstate feeds url
                          (press Enter)
3) Provide the path of sstate-cache from above
           Ex: /<path>/aarch64  for Versal projects

4) Save the configurations to use the sstate-cache

Setting download mirror

1) Extract the downloaded tar contents of downloads at any location
2) run petalinux-config
        -> Yocto Settings
                -> Add pre-mirror url
						          ->  (press Enter)
							            Clear default value

3) Provide the path of downloads from above
        file://<path>/downloads for all projects

4) Save the configurations to use the download mirrors

5) Verify the changes in build/conf/plnxtool.conf
6) petalinux-build
```


### 2019.1
```bash
By default petalinux uses sstate-cache and download mirrors from petalinux.xilinx.com
This README is for the users who uses tool completely offline.

Setting sstate cache

1) Extract the downloaded tar contents of sstate-cache at any location
2) run petalinux-config
         -> Yocto Settings
             ->Local sstate feeds settings
                  ->local sstate feeds url
                          (press Enter)
3) Provide the path of sstate-cache from above
           Ex: /<path>/aarch64  for ZynqMP projects
               /<path>/arm      for Zynq projects
               /<path>/mb-full  for MB AXI full projects

4) Save the configurations to use the sstate-cache

Setting download mirror

1) Extract the downloaded tar contents of sstate-cache at any location
2) run petalinux-config
        -> Yocto Settings
                -> Add pre-mirror url
						          ->  (press Enter)
							            Clear default value

3) Provide the path of sstate-cache from above
        file://<path>/downloads for all projects

4) Save the configurations to use the download mirrors

5) Verify the changes in build/conf/plnxtool.conf
6) petalinux-build
```

---

## Comments

### konosubakonoakua · 2024-09-25T17:02:13Z

## nfs for sstate-cache
You should have `/tools/petalinux/sstate-cache` at this time.
```bash
sudo cp -f /etc/exports /etc/exports.bak
sudo tee -a /etc/exports << EOF
/tools/petalinux/sstate-cache *(rw,sync,no_root_squash)
EOF
```
restart and show
```bash
sudo service nfs-kernel-server restart
showmount -e
```
find your ip address
```bash
hostname -I
```


### konosubakonoakua · 2024-09-25T18:26:25Z

## SSTATE_DIR & DL_DIR
can I map these two to sstate folder? (tbd)

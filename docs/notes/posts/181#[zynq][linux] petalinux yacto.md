---
number: 181
title: "[zynq][linux] petalinux yacto"
state: open
url: https://github.com/konosubakonoakua/blog/issues/181
author: konosubakonoakua
createdAt: 2025-05-13T08:42:46Z
updatedAt: 2025-05-13T08:42:50Z
labels: []
---

# 181 [zynq][linux] petalinux yacto

## install apps to another location
```bash
petalinux-create -t apps --template <type> --name <app-name> --enable
```
```bb
do_install_append() {
    install -d ${TOPDIR}/../images/linux/apps/
    install -m 0755 ${D}${bindir}/<appname> ${TOPDIR}/../images/linux/apps/
}
```
## install modules to another location
```bash
MODULES_DIR="${TOPDIR}/../images/linux/modules"

do_install_append() {
    mkdir -p ${MODULES_DIR}
    cp ${B}/zynq-irq.ko ${MODULES_DIR}
}
```

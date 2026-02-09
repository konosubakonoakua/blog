---
number: 173
title: "[win] surface restore"
state: open
url: https://github.com/konosubakonoakua/blog/issues/173
author: konosubakonoakua
createdAt: 2025-04-21T11:10:34Z
updatedAt: 2025-04-22T02:45:21Z
labels: []
---

# 173 [win] surface restore



## UEFI setting (`Power` + `Volume Up`)
- must set u-disk as the first boot option

## surface-recovery-image
- https://support.microsoft.com/zh-cn/surface-recovery-image
- you need your surface id

## create recovery drive
- use `Recovery Drive`, **DONOT** select `Back up system files to the recovery drive.`
- wait for complete, the u-disk should have 32GB free space or more

## boot to udisk
- plug in udisk, reboot

## restore new system
- use `restore from udisk`

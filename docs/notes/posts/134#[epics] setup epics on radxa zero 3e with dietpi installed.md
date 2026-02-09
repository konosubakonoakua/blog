---
number: 134
title: "[epics] setup epics on radxa zero 3e with dietpi installed"
state: open
url: https://github.com/konosubakonoakua/blog/issues/134
author: konosubakonoakua
createdAt: 2025-01-20T14:12:28Z
updatedAt: 2025-01-21T06:54:40Z
labels: []
---

# 134 [epics] setup epics on radxa zero 3e with dietpi installed

## epoll issue (dietpi)
After I installed softioc, encounterd:
```
libevent Warn: epoll_wait: Function not implemented
2025-01-20T14:12:52.354942535 CRIT pvxs.loop Exit loop worker: -1 for 0xffffa8000c70
```
must add `EVENT_NOEPOLL=1` before your command
or just insert the code below
```python
import os
os.environ['EVENT_NOEPOLL'] = '1'
```
I have raised an issue in pvxs: https://github.com/epics-base/pvxs/issues/96

---

## Comments

### konosubakonoakua · 2025-01-20T16:34:25Z

## i2c issue
> i2c3 on Pin 3/5 is not enabled by default.
> BTW 0x22 is occupied by PD delivery chip

We need to use additional device tree overlays [here(radxa-overlays action artifacts)](https://github.com/radxa-pkg/radxa-overlays/actions/runs/12763021517)
Or just use the file I have uploaded: [linux-rk356x.zip](https://github.com/user-attachments/files/18480217/linux-rk356x.zip)

Then
```bash
cp ~/shared/rk3568-i2c3-m0.dtbo /boot/dtb-5.10.160-legacy-rk35xx/rockchip/
```
Then modify `/boot/dietpiEnv.txt`
```bash
overlay_path=rockchip
overlay_prefix=rk3568
overlays=i2c3-m0
user_overlays=
```


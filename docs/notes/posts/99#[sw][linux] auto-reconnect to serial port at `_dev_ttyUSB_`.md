---
number: 99
title: "[sw][linux] auto-reconnect to serial port at `/dev/ttyUSB*`"
state: open
url: https://github.com/konosubakonoakua/blog/issues/99
author: konosubakonoakua
createdAt: 2024-09-25T15:26:28Z
updatedAt: 2024-09-25T22:45:38Z
labels: ["Linux"]
---

# 99 [sw][linux] auto-reconnect to serial port at `/dev/ttyUSB*`

> Thanks to [lundmar](https://github.com/tio/tio/issues/279), I have a new solution. :D
## install tio
```bash
snap install tio --classic
```
or check it [here](https://github.com/tio/tio#4-installation)
## connect it
```bash
tio -a latest --exclude-drivers=!(cp210x)
```

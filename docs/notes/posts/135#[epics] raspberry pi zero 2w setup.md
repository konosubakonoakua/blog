---
number: 135
title: "[epics] raspberry pi zero 2w setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/135
author: konosubakonoakua
createdAt: 2025-01-21T09:36:09Z
updatedAt: 2025-02-10T08:28:32Z
labels: []
---

# 135 [epics] raspberry pi zero 2w setup

## image download
[raspberry-pi-os-64-bit](https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit)
## ENABLE UART in `config.txt`
> pinout [here](https://pinout.xyz/)
> UART-TX-GPIO14
> UART-RX-GPIO15

Add `enable_uart=1` in `config.txt`. Refer to [here](https://www.raspberrypi.com/documentation/computers/config_txt.html#enable_uart)


---

## Comments

### konosubakonoakua · 2025-02-10T06:53:20Z

## pcaspy
```bash
export PIP_NO_USE_PEER_RETRIES=1 # if mem < 1G
uv pip install pvxslibs epicsdbbuilder pyyaml cothread
uv pip install softioc --no-deps
```

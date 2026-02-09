---
number: 68
title: "[dev][env][linux][linuxmint] linuxmint setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/68
author: konosubakonoakua
createdAt: 2024-06-29T07:02:23Z
updatedAt: 2024-07-09T07:25:33Z
labels: []
---

# 68 [dev][env][linux][linuxmint] linuxmint setup

## iso download
- https://mirrors.bfsu.edu.cn/linuxmint-cd/stable/21.3/
## sources list
- **现在推荐直接使用自带的工具，还能测试各个源的速度**
- https://mirrors.bfsu.edu.cn/help/linuxmint/

---

## Comments

### konosubakonoakua · 2024-07-09T07:25:33Z

## firewall
### opensnitch
- https://github.com/evilsocket/opensnitch/wiki/Installation

#### cannot show GUI
```shell
# as user (no sudo)
pip install --upgrade pip
pip install grpcio-tools grpcio protobuf slugify qt-material pyasn
pip install --user --ignore-installed grpcio==1.44.0 grpcio-tools==1.44.0
	# downgrade grpc version https://github.com/evilsocket/opensnitch/issues/647


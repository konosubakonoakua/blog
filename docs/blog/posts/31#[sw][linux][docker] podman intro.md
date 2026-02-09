---
number: 31
title: "[sw][linux][docker] podman intro"
state: open
url: https://github.com/konosubakonoakua/blog/issues/31
author: konosubakonoakua
createdAt: 2024-04-26T09:03:38Z
updatedAt: 2024-04-26T09:17:41Z
labels: []
---

# 31 [sw][linux][docker] podman intro

## installation
- https://podman.io/docs/installation
- https://github.com/containers/podman/releases

## issues
- [insufficient UIDs or GIDs in user namespace](https://github.com/containers/podman/issues/12715#issuecomment-1207559582)
```bash
sudo usermod --add-subgids 10000-75535 USERNAME
sudo usermod --add-subuids 10000-75535 USERNAME
podman system migrate
podman pull ...
```
- [placeholder]()

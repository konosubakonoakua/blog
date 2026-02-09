---
number: 86
title: "[sw][linux] change source mirror registry 换源"
state: open
url: https://github.com/konosubakonoakua/blog/issues/86
author: konosubakonoakua
createdAt: 2024-09-21T19:57:16Z
updatedAt: 2024-11-27T01:51:32Z
labels: ["Linux", "python"]
---

# 86 [sw][linux] change source mirror registry 换源

## mirrors
- https://mirrors.bfsu.edu.cn/help/
## python
```bash
pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple
```
## rust
```bash
export CARGO_REGISTRIES_CRATES_IO_PROTOCOL=git
export CARGO_REGISTRIES_CRATES_IO_INDEX=https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git
```

---

## Comments

### konosubakonoakua · 2024-09-21T20:15:30Z

## docker
- https://jockerhub.com/
- https://docker.1panel.dev/
```bash
sudo tee /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": ["https://docker.1panel.dev"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### konosubakonoakua · 2024-09-21T20:15:39Z

## npm
```bash
npm config set registry https://registry.npmmirror.com
# https://npmmirror.com/
```

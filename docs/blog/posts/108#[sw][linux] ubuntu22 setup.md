---
number: 108
title: "[sw][linux] ubuntu22 setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/108
author: konosubakonoakua
createdAt: 2024-10-04T15:20:14Z
updatedAt: 2024-10-19T01:15:56Z
labels: []
---

# 108 [sw][linux] ubuntu22 setup

## can not update snap store
Please update manually.
```bash
sudo killall snap-store
sudo snap refresh snap-store
```

---

## Comments

### konosubakonoakua · 2024-10-04T15:25:06Z

## Microsoft edge install
All linux version edges are listed [here](https://packages.microsoft.com/repos/edge/pool/main/m/microsoft-edge-stable/)
```bash
sudo apt install <your edge deb file path>
```

### konosubakonoakua · 2024-10-19T01:15:55Z

## nodejs install
### use `nvm`
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```
Install nodejs 18 version
```bash
nvm install 18
nvm use 18
```

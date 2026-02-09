---
number: 111
title: "[sw][scoop][windows] scoop usage"
state: open
url: https://github.com/konosubakonoakua/blog/issues/111
author: konosubakonoakua
createdAt: 2024-10-08T08:11:15Z
updatedAt: 2024-10-08T08:20:43Z
labels: []
---

# 111 [sw][scoop][windows] scoop usage

[scoop](https://scoop.sh/)

---

## Comments

### konosubakonoakua · 2024-10-08T08:20:41Z

## Install scoop itself
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```
## Install git/7zip/aria2/... base packages
```powershell
scoop install sudo
scoop install 7zip git
scoop install openssh aria2 curl grep sed less touch
scoop config aria2-warning-enabled false
```
## Add buckets
```powershell
scoop bucket known
scoop bucket add extras
scoop bucket add versions
# ...
```
## search for package
```
scoop search <keyword>
scoop install ...
```

## clean up all
```powershell
scoop cache rm -a && scoop cleanup *
```

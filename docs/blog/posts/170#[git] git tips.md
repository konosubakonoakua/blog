---
number: 170
title: "[git] git tips"
state: open
url: https://github.com/konosubakonoakua/blog/issues/170
author: konosubakonoakua
createdAt: 2025-04-10T02:57:47Z
updatedAt: 2025-07-11T08:46:32Z
labels: []
---

# 170 [git] git tips

## introduction

https://rogerdudler.github.io/git-guide/index.zh.html

---

## Comments

### konosubakonoakua · 2025-04-13T20:01:40Z

## handle dos whitespaces
```bash
git config --global core.autocrlf input
```

### konosubakonoakua · 2025-04-18T06:00:08Z

### `file new modes` when cloning linux repo to windows
tell git to ignore permission change
```bash
git config core.filemode false
```

### konosubakonoakua · 2025-04-22T09:39:17Z

## lfs
```bash
git lfs install
git lfs pull
```

### konosubakonoakua · 2025-07-11T08:46:32Z

## bad object refs/tags/xxx
```bash
rm .git/refs/tags/xxx
```

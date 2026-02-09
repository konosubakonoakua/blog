---
number: 88
title: "[sw][linux] utils tools"
state: open
url: https://github.com/konosubakonoakua/blog/issues/88
author: konosubakonoakua
createdAt: 2024-09-22T00:00:52Z
updatedAt: 2024-09-25T22:50:52Z
labels: ["Linux"]
---

# 88 [sw][linux] utils tools

## just
```bash
mkdir -p ~/.local/bin
curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to ~/.local/bin
export PATH="$PATH:$HOME/.local/bin"
just --help
```

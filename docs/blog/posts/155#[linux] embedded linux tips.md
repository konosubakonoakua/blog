---
number: 155
title: "[linux] embedded linux tips"
state: open
url: https://github.com/konosubakonoakua/blog/issues/155
author: konosubakonoakua
createdAt: 2025-02-27T02:20:20Z
updatedAt: 2025-03-25T09:23:29Z
labels: []
---

# 155 [linux] embedded linux tips

## Send bytes to tty device using bash
```bash
stty -F /dev/ttyPS1
stty -F /dev/ttyPS1 115200 cs8 -cstopb -parenb
printf '\xA5' | hexdump -C

printf '\xA5' | cat > /dev/ttyPS1
```

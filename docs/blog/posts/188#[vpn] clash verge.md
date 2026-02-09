---
number: 188
title: "[vpn] clash verge"
state: open
url: https://github.com/konosubakonoakua/blog/issues/188
author: konosubakonoakua
createdAt: 2025-06-05T09:12:52Z
updatedAt: 2025-06-05T09:12:56Z
labels: []
---

# 188 [vpn] clash verge

## lan firewall
```bash
netsh advfirewall firewall add rule name="verge-mihomo" dir=in action=allow protocol=UDP localport=7897
netsh advfirewall firewall add rule name="verge-mihomo" dir=in action=allow protocol=TCP localport=7897
```

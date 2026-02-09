---
number: 91
title: "[sw][linux][sys][network] linux network setting"
state: open
url: https://github.com/konosubakonoakua/blog/issues/91
author: konosubakonoakua
createdAt: 2024-09-23T01:59:01Z
updatedAt: 2024-09-29T02:14:02Z
labels: ["Linux"]
---

# 91 [sw][linux][sys][network] linux network setting

```bash
sudo apt install net-tools
ipconfig
```

```bash
sudo ip link set dev eno1 down
sudo ip link set dev eno1 address XX:XX:XX:XX:XX:XX
sudo ip link set dev eno1 up
```


---

## Comments

### konosubakonoakua · 2024-09-26T03:43:00Z

## references
- https://www.baeldung.com/linux/set-static-ip-address
- https://www.debian.org/doc/manuals/debian-reference/ch05.en.html#_the_hostname_resolution

### konosubakonoakua · 2024-09-27T09:57:54Z

## change hostname
On Debian system, [here's the doc](https://www.debian.org/doc/manuals/debian-reference/ch05.en.html#_the_hostname_resolution).
```bash
_HOSTNAME=RF-PC-TS-LINUX
sudo hostname $_HOSTNAME
sudo sed -i -E "s#(127\.0\.1\.1\s+)(.*)#\1$_HOSTNAME#" /etc/hosts
```

---
number: 90
title: "[sw][linux][command] tar multithread"
state: open
url: https://github.com/konosubakonoakua/blog/issues/90
author: konosubakonoakua
createdAt: 2024-09-22T03:59:52Z
updatedAt: 2024-09-25T22:51:48Z
labels: ["Linux"]
---

# 90 [sw][linux][command] tar multithread

## tar.xz
```bash
tar -cvf - my_folder/ | xz -T 0 -c > my_folder.tar.xz
```
-T 后面跟上的参数是要使用的线程数量，0 代表尽可能多的使用 CPU 线程。

---

## Comments

### konosubakonoakua · 2024-09-22T04:02:11Z

## pigz
compress
```bash
tar --use-compress-program="pigz -k -p8" -cf output.tgz *
```
decompress (a little bit faster)
```bash
tar --use-compress-program="pigz -k -p8" -xf output.tgz
```


### konosubakonoakua · 2024-09-22T04:14:05Z

## references
- https://unix.stackexchange.com/questions/608207/how-to-use-multi-threading-for-creating-and-extracting-tar-xz

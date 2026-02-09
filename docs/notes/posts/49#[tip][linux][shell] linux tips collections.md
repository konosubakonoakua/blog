---
number: 49
title: "[tip][linux][shell] linux tips collections"
state: open
url: https://github.com/konosubakonoakua/blog/issues/49
author: konosubakonoakua
createdAt: 2024-05-14T07:31:39Z
updatedAt: 2024-05-24T06:17:08Z
labels: []
---

# 49 [tip][linux][shell] linux tips collections

## Dynamic variable names in Bash
```bash
#!/bin/sh
# In the container at runtime:
# export PROD_TOKEN="123"
env="PROD"
secret="${env}_TOKEN"
echo ${!secret}
```

---

## Comments

### konosubakonoakua · 2024-05-14T07:32:29Z

## Shell-Parameter-Expansion
https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html

### konosubakonoakua · 2024-05-14T08:04:25Z

## install location
- cmake
        ```cmake
        cmake -DCMAKE_INSTALL_PREFIX=$HOME/.local
        ```
- configure
        ```bash
        ./configure --prefix=$HOME/.local
        ```

### konosubakonoakua · 2024-05-14T08:23:52Z

## LD_LIBRARY_PATH

```
Libraries have been installed in:
   /home/USER/.local/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.

```

```bash
❯ cat /etc/ld.so.conf.d/x86_64-linux-gnu.conf 
# Multiarch support
/usr/local/lib/x86_64-linux-gnu
/lib/x86_64-linux-gnu
/usr/lib/x86_64-linux-gnu

```

```bash

echo
echo '== env =========================================================='
if [ -z ${LD_LIBRARY_PATH+x} ]; then
    echo "LD_LIBRARY_PATH is unset";
else
    echo LD_LIBRARY_PATH ${LD_LIBRARY_PATH} ;
fi

if [ -z ${LD_RUN_PATH+x} ]; then
    echo "LD_RUN_PATH is unset";
else
    echo LD_RUN_PATH ${LD_RUN_PATH} ;
fi

if [ -z ${DYLD_LIBRARY_PATH+x} ]; then
    echo "DYLD_LIBRARY_PATH is unset";
else
    echo DYLD_LIBRARY_PATH ${DYLD_LIBRARY_PATH} ;
fi
echo '================================================================='
echo
echo
 
```

### konosubakonoakua · 2024-05-24T06:14:21Z

## float point number comparison
use `bc` like `echo "$1 < $2" | bc`
## ripgrep output only capture group
```shell
# only output capture group 1
rg '(group 1)...(group 2)' -or '$1'
```
## libc version check
```shell
libc_ver_lt () {
  bc_cmd="$(ldd --version | rg 'GLIBC (\d\.\d{2})' -or '$1') < $1"
  return $(echo "$bc_cmd" | bc)
}
```

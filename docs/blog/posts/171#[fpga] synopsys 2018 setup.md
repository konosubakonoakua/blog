---
number: 171
title: "[fpga] synopsys 2018 setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/171
author: konosubakonoakua
createdAt: 2025-04-18T06:07:15Z
updatedAt: 2025-04-19T11:17:37Z
labels: []
---

# 171 [fpga] synopsys 2018 setup

## deps
```bash
 ## for lmli
sudo apt-get install lsb

## for verdi
sudo wget http://se.archive.ubuntu.com/ubuntu/pool/main/libp/libpng/libpng12-0_1.2.54-1ubuntu1_amd64.deb
sudo dpkg -i libpng12-0_1.2.54-1ubuntu1_amd64.deb
sudo apt-get install libncurses5 -y
```

## mkdir
```bash
sudo mkdir -p /tools
sudo chown user:user /tools

# for lmli
sudo mkdir -p /usr/tmp
sudo chmod 777 /usr/tmp/
```

## install
```bash
cd synopsysinstaller_v5.0/
chmod +x ./SynopsysInstaller_v5.0.run
./SynopsysInstaller_v5.0.run # install in current dir
./setup.sh # select installers folders
```

## keygen
- https://blog.csdn.net/lum250/article/details/123755261
  - `EXPIRE` must be `12-dec-2030`
  - `HOST ID Daemon/Feature` is MAC Adress without `:`
  - HOSTNAME is `$HOSTNAME`
  - modify license file: `DAEMON snpslmd /tools/Synopsis/2018/scl/2018.06/linux64/bin/snpslmd`
 

## env
```bash
export SYNOPSYS=/tools/Synopsys/2018
export DVE_HOME=$SYNOPSYS/vcs-mx/O-2018.09-SP2/gui/dve
export VCS_HOME=$SYNOPSYS/vcs-mx/O-2018.09-SP2
export VERDI_HOME=$SYNOPSYS/verdi/Verdi_O-2018.09-SP2
export NOVAS_HOME=$VERDI_HOME
export SCL_HOME=$SYNOPSYS/scl/2018.06
#export LD_LIBRARY_PATH="$VERDI_HOME/share/VCS/lib/LINUX64":$LD_LIBRARY_PATH
export LD_LIBRARY_PATH="$VERDI_HOME/share/PLI/lib/LINUX64":$LD_LIBRARY_PATH
#export LD_LIBRARY_PATH="$NOVAS_HOME/share/NPI/lib/LINUX64_GNU_520":$LD_LIBRARY_PATH
export PATH=$PATH:$DVE_HOME/bin:$VCS_HOME/bin:$VERDI_HOME/bin:$SCL_HOME/linux64/bin

export VCS_TARGET_ARCH="amd64"
export VCS_ARCH_OVERRIDE=linux
export NPI_PLATFORM="LINUX64_GNU_472"
export LM_LICENSE_FILE="27000@$HOSTNAME"

alias dve='dve -full64'
alias vcs='vcs'
alias verdi='verdi'
alias lmg_vcs='lmgrd -c $SCL_HOME/admin/license/Synopsys.dat'
```


## license test
```bash
kill -9 $(lsof -ti :27000)
lmg_vcs
```
## autostart
```bash
sudo tee /etc/profile.d/synopsys-lmli.sh << EOF
export SYNOPSYS=/tools/Synopsys/2018
$SYNOPSYS/scl/2018.06/linux64/bin/lmgrd -c $SYNOPSYS/Liscen/Synopsys.dat -l $SYNOPSYS/scl/2018.06/linux64/bin/lmgrd.log
EOF
```

## troubleshoot
### uvm simple cannot run, need modification
```makefile
# /tools/Synopsys/2018/vcs-mx/O-2018.09-SP2/doc/examples/testbench/sv/uvm/simple/hello_world/Makefile

all: clean comp run

clean:
	rm -rf simv* csrc *.log vc_hdrs.h ucli.key

comp:
	vcs -full64 -cpp g++ -cc gcc -LDFLAGS -no-pie -LDFLAGS -Wl,--no-as-needed -CFLAGS -fPIE -sverilog -timescale=1ns/1ns -ntb_opts uvm-1.1 +incdir+. hello_world.sv -l comp.log

run:
	./simv  +UVM_NO_RELNOTES  -l run.log
```

### verdi libtinfo.so.5 not found
```bash
sudo mv $SYNOPSYS/verdi/Verdi_O-2018.09-SP2/etc/lib/libstdc++/LINUXAMD64/libtinfo.so.5 $SYNOPSYS/verdi/Verdi_O-2018.09-SP2/etc/lib/libstdc++/LINUXAMD64/libtinfo.so.5.bak
sudo ln -s /lib/x86_64-linux-gnu/libtinfo.so.5 $SYNOPSYS/verdi/Verdi_O-2018.09-SP2/etc/lib/libstdc++/LINUXAMD64/libtinfo.so.5
```

### vcs: linux kernel xxx not supported
need to add `-full64` & set `export VCS_ARCH_OVERRIDE=linux`.

### /bin/sh: Illegal option -h​
switch to bash
```bash
sudo dpkg-reconfigure dash
```

### lmgrd: No such file or directory
```bash
sudo apt install lsb-core
```

### ​license daemon: system error code: No such file or directory​
```bash
kill -9 $(lsof -ti :27000)
```

### /libvcsnew.so: undefined reference to xxx​
Caution, there's no space between **-Wl,** & **--no-as-needed**
```makefile
 -LDFLAGS -Wl,--no-as-needed
```
Recommended: `-full64 -cpp g++-4.8 -cc gcc-4.8 -LDFLAGS -Wl,--no-as-needed`

### vcs: line 3312: dc: command not found​
`sudo apt install dc`

### error while loading shared libraries: libpng12.so.0
```bash
wget http://security.ubuntu.com/ubuntu/pool/main/libp/libpng/libpng12-0_1.2.54-1ubuntu1.1_amd64.deb
sudo dpkg -i ./libpng12-0_1.2.54-1ubuntu1.1_amd64.deb
```

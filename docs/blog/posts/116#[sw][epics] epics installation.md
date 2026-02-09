---
number: 116
title: "[sw][epics] epics installation"
state: open
url: https://github.com/konosubakonoakua/blog/issues/116
author: konosubakonoakua
createdAt: 2024-10-22T07:44:11Z
updatedAt: 2026-01-21T03:26:29Z
labels: ["Linux", "EPICS"]
---

# 116 [sw][epics] epics installation

<details>

<summary> Epics Base </summary>

## Epics Base
Dependencies
```bash
sudo apt install build-essential libreadline-dev perl tcl tk
```
Make Directories
```bash
sudo mkdir -p /epics
sudo chown $USER:$USER /epics
ln -s /epics $HOME/EPICS
cd /epics
```

Clone Repo
```bash
wget https://epics-controls.org/download/base/base-7.0.8.1.tar.gz
tar -xvf base-7.0.8.1.tar.gz
cd base-7.0.8.1
```
Or,
```bash
git clone --recursive -b 7.0 https://git.launchpad.net/epics-base base-7.0
cd base-7.0
```
Or,
```bash
# my fork
git clone --recursive -b 7.0 https://github.com/konosubakonoakua/epics-base base-7.0
cd base-7.0
```
Make
```bash
make -j8
```
If failed, and need to rebuild
```bash
make distclean
git reset --hard
git clean -fd
```
If need to checkout to another tag:
```bash
# after checkout
make distclean
git reset --hard
git clean -fd
rm -rf .ci
rm -rf ./modules
git submodule update --init --recursive
```
Setup Env (Optional)
```bash
cat >> ~/.bashrc << 'EOF'

export EPICS_BASE=/epics/3.15/base
export EPICS_HOST_ARCH=$(${EPICS_BASE}/startup/EpicsHostArch)
export PATH=${EPICS_BASE}/bin/${EPICS_HOST_ARCH}:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:${EPICS_BASE}/lib/${EPICS_HOST_ARCH}

EOF
```
Test
```bash
which softIoc
softIoc
```
Test softIoc app
```bash
mkdir -p $HOME/EPICS/iocs/testIoc
cd $HOME/EPICS/iocs/testIoc
makeBaseApp.pl -t example testIoc
makeBaseApp.pl -i -t example testIoc
make
cd iocBoot/ioctestIoc
chmod +x ./st.cmd
./st.cmd
```

<details>

---

## Comments

### konosubakonoakua · 2024-10-23T03:20:50Z

<details>

<summary> Download synApps </summary>

### Download synApps
> Consider using `assemble_synApps.sh`

First check the latest version: https://github.com/EPICS-synApps/support/releases/latest
For now, it's `R6-3`
```bash
export _SYNAPPS_VERSION=R6-3
export _SYNAPPS_FILENAME=synApps_6_3
export _SYNAPPS=/epics/$_SYNAPPS_FILENAME
cd ~/Downloads
wget https://github.com/EPICS-synApps/support/releases/download/$_SYNAPPS_VERSION/$_SYNAPPS_FILENAME.tar.gz
if [ $? -eq 0 ]; then
  tar -xzf $_SYNAPPS_FILENAME.tar.gz -C /epics/
  rm $_SYNAPPS_FILENAME.tar.gz
  cd /epics
fi
```
#### Copy desired modules
```bash
export _EPICS_VERSION=3.15
export _EPICS_BASE=/epics/$_EPICS_VERSION/base
export _EPICS_SUPPORT=/epics/$_EPICS_VERSION/support
mkdir -p $_EPICS_SUPPORT
cd $_EPICS_SUPPORT
cp -r $_SYNAPPS/support/StreamDevice* .
cp -r $_SYNAPPS/support/asyn* .
cp -r $_SYNAPPS/support/calc* .
cp -r $_SYNAPPS/support/sscan* .
cp -r $_SYNAPPS/support/sequencer-mirror* .
```

### Modify support modules (not tested yet)
```bash
cd $_EPICS_SUPPORT
export REPO_SEQUENCER_MIRROR=$(find $_EPICS_SUPPORT -maxdepth 1 -type d -name "*sequencer-mirror*" -exec basename {} \;)
export REPO_SSCAN=$(find $_EPICS_SUPPORT -maxdepth 1 -type d -name "*sscan*" -exec basename {} \;)
export REPO_CALC=$(find $_EPICS_SUPPORT -maxdepth 1 -type d -name "*calc*" -exec basename {} \;)
export REPO_ASYN=$(find $_EPICS_SUPPORT -maxdepth 1 -type d -name "*asyn*" -exec basename {} \;)
export REPO_STREAMDEVICE=$(find $_EPICS_SUPPORT -maxdepth 1 -type d -name "*StreamDevice*" -exec basename {} \;)
```

#### StreamDevice
```bash
tee > ${_EPICS_SUPPORT}/${REPO_STREAMDEVICE}/configure/RELEASE.local << EOF
SUPPORT=${_EPICS_SUPPORT}
ASYN=\$(SUPPORT)/${REPO_ASYN}
CALC=\$(SUPPORT)/${REPO_CALC}
PCRE=
EPICS_BASE=${_EPICS_BASE}
EOF
```
```bash
rm -f ${_EPICS_SUPPORT}/${REPO_STREAMDEVICE}/GNUmakefile
```

#### Asyn

```bash
tee > ${_EPICS_SUPPORT}/${REPO_ASYN}/configure/RELEASE.local << EOF
SUPPORT=${_EPICS_SUPPORT}
SNCSEQ=\$(SUPPORT)/${REPO_SEQUENCER_MIRROR}
CALC=\$(SUPPORT)/${REPO_CALC}
SSCAN=\$(SUPPORT)/${REPO_SSCAN}
EPICS_BASE=${_EPICS_BASE}
EOF
```
```bash
tee > ${_EPICS_SUPPORT}/${REPO_ASYN}/configure/CONFIG_SITE.local << EOF
TIRPC=YES
EOF
```
If TIRPC compilation failed, refer to https://github.com/konosubakonoakua/blog/issues/92#issuecomment-2471131689 for multiarch package.

#### CALC

```bash
tee > ${_EPICS_SUPPORT}/${REPO_CALC}/configure/RELEASE.local << EOF
SUPPORT=${_EPICS_SUPPORT}
SSCAN=\$(SUPPORT)/${REPO_SSCAN}
SNCSEQ=\$(SUPPORT)/${REPO_SEQUENCER_MIRROR}
CALC=\$(SUPPORT)/${REPO_CALC}
EPICS_BASE=${_EPICS_BASE}
EOF
```

#### SSCAN

```bash
tee > ${_EPICS_SUPPORT}/${REPO_SSCAN}/configure/RELEASE.local << EOF
SUPPORT=${_EPICS_SUPPORT}
SNCSEQ=\$(SUPPORT)/${REPO_SEQUENCER_MIRROR}
EPICS_BASE=${_EPICS_BASE}
EOF
```

#### SEQUENCER_MIRROR

```bash
tee > ${_EPICS_SUPPORT}/${REPO_SEQUENCER_MIRROR}/configure/RELEASE.local << EOF
EPICS_BASE=${_EPICS_BASE}
EOF
```

### Compilation order (EPICS_BASE must have been compiled)
```bash
cd ${_EPICS_SUPPORT}/${REPO_SEQUENCER_MIRROR} && make distclean && make -j4
```
```bash
cd ${_EPICS_SUPPORT}/${REPO_SSCAN} && make distclean && make -j4
```
```bash
cd ${_EPICS_SUPPORT}/${REPO_CALC} && make distclean && make -j4
```
```bash
cd ${_EPICS_SUPPORT}/${REPO_ASYN} && make distclean && make -j4
```
```bash
cd ${_EPICS_SUPPORT}/${REPO_STREAMDEVICE} && make distclean && make -j4
```

</details>

<details>

<summary>
If you only want to try Asyn & StreamDevice
</summary>

## Asyn
Make dir
```bash
cd ${EPICS_BASE}
mkdir -p support
cd support
```
Clone
```bash
git clone https://github.com/epics-modules/asyn.git
cd asyn/configure
```
Modify
Use these command for dry-run, only checking
```bash
sed -n "s|EPICS_BASE=.*|EPICS_BASE=${EPICS_BASE}|p" RELEASE
sed -n "s|SUPPORT=.*|SUPPORT=${EPICS_BASE}/support|p" RELEASE
sed -n "s|# TIRPC=YES|TIRPC=YES|p" CONFIG_SITE
```
Really modify
```bash
sed -i "s|EPICS_BASE=.*|EPICS_BASE=${EPICS_BASE}|" RELEASE
sed -i "s|SUPPORT=.*|SUPPORT=${EPICS_BASE}/support|" RELEASE
sed -i "s|# TIRPC=YES|TIRPC=YES|" CONFIG_SITE
```
Make it
```bash
make -j8
```

## StreamDevice
Make dir
```bash
mkdir -p ${EPICS_BASE}/support
```
Clone
```bash
cd ${EPICS_BASE}/support
git clone https://github.com/paulscherrerinstitute/StreamDevice.git
cd StreamDevice/
rm GNUmakefile
```
Modify
Use these command for dry-run, only checking
```bash
sed -n "s|EPICS_BASE=.*|EPICS_BASE=${EPICS_BASE}|p" RELEASE
sed -n "s|ASYN=.*|ASYN=\$(SUPPORT)/asyn|p" RELEASE
sed -n "/CALC=.*/s|^|# |p" RELEASE
sed -n "/PCRE=.*/s|^|# |p" RELEASE
```
Really modify
We don't have CALC & PCRE support installed now, so disable them.
```bash
sed -i "s|EPICS_BASE=.*|EPICS_BASE=${EPICS_BASE}|" RELEASE
sed -i "s|ASYN=.*|ASYN=\$(SUPPORT)/asyn|" RELEASE
sed -i "/CALC=.*/s|^|# |" RELEASE
sed -i "/PCRE=.*/s|^|# |" RELEASE
```
Make it
```bash
make -j8
```
If we want to use CALC, then I recommend installing synApps.

</details>

### konosubakonoakua · 2024-10-28T06:19:37Z

<details>

<summary> cross compile </summary>

## cross-compile
### Install compiler for `zynq cortex-a9`
#### ~arm-linux-gnueabi~ (ld-linux.so.3 not found!!)
```bash
sudo apt install gcc-arm-linux-gnueabi g++-arm-linux-gnueabi binutils-arm-linux-gnueabi
```
#### arm-linux-gnueabihf (ok, but need matched GBLIC version on target & host)
```bash
sudo apt install gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf binutils-arm-linux-gnueabihf
```
#### arm-linux-gnueabihf (Xilinx SDK provided, totally ok)
```bash
ls /tools/Xilinx/SDK/2019.1/gnu/aarch32/lin/gcc-arm-linux-gnueabi
```
#### aarch64-linux-gnu
```bash
sudo apt install  gcc-aarch64-linux-gnu g++-aarch64-linux-gnu binutils-aarch64-linux-gnu
```
### Install `qemu-user-static` (for running arm executables)
```bash
sudo apt install qemu-user-static qemu-user
```
### Install dependencies
```bash
export EPICS_TOOLS_ARM_PKGS=/epics/tools/arm/pkgs
mkdir -p $EPICS_TOOLS_ARM_PKGS
```
#### ~Install `readline`~
```bash
cd $EPICS_TOOLS_ARM_PKGS
wget https://mirrors.ustc.edu.cn/gnu/readline/readline-8.2.tar.gz
tar -xvf readline-8.2.tar.gz
cd $EPICS_TOOLS_ARM_PKGS/readline-8.2
```
```bash
export _EPICS_TARGET_ARCH=arm-linux-gnueabihf
CC=${_EPICS_TARGET_ARCH}-gcc ./configure --host=${_EPICS_TARGET_ARCH} --prefix=/usr/${_EPICS_TARGET_ARCH}
make -j4
sudo make install
```
#### ~Install `ncurses`~
```bash
cd $EPICS_TOOLS_ARM_PKGS
wget https://mirrors.ustc.edu.cn/gnu/ncurses/ncurses-6.5.tar.gz
tar -xvf ncurses-6.5.tar.gz
cd $EPICS_TOOLS_ARM_PKGS/ncurses-6.5
```
```bash
export _EPICS_TARGET_ARCH=arm-linux-gnueabihf
# NOTE: Beginning with ncurses 6.5 option `--enable-widec` is enabled by default; Older versions disable it by default.
CC=${_EPICS_TARGET_ARCH}-gcc ./configure --host=${_EPICS_TARGET_ARCH} --prefix=/usr/${_EPICS_TARGET_ARCH} --with-normal --with-shared --with-strip-program=${_EPICS_TARGET_ARCH}-strip  --disable-widec --without-tests --enable-pc-files --with-pkg-config --with-pkg-config-libdir=libdir
make -j4
sudo make install
```
use armhf package instead, details [here](https://github.com/konosubakonoakua/blog/issues/92#issuecomment-2471131689).
```bash
sudo dpkg --add-architecture i386
sudo dpkg --add-architecture armhf
sudo dpkg --add-architecture arm64
sudo dpkg --print-foreign-architectures
sudo apt install -y --no-install-recommends libc6:armhf libstdc++6:armhf libreadline-dev:armhf libncurses-dev:armhf

```
### Compile epics-base
#### Modify compiler & libraries options
```bash
export EPICS_BASE=/epics/3.15/base
export EPICS_BASE=/epics/7/base
cd $EPICS_BASE
```
##### Use patches
```diff
git apply << 'EOF'
diff --git a/configure/os/CONFIG_SITE.linux-x86.linux-arm b/configure/os/CONFIG_SITE.linux-x86.linux-arm
index 231c54bb8..b257582b1 100644
--- a/configure/os/CONFIG_SITE.linux-x86.linux-arm
+++ b/configure/os/CONFIG_SITE.linux-x86.linux-arm
@@ -4,12 +4,14 @@
 #-------------------------------------------------------
 
 # Set GNU crosscompiler target name
-GNU_TARGET = arm-xilinx-linux-gnueabi
+GNU_TARGET = arm-linux-gnueabihf
 
 # Set GNU tools install path
 # Examples are installations at the APS:
+#GNU_DIR = /usr
+GNU_DIR = /tools/Xilinx/SDK/2019.1/gnu/aarch32/lin/gcc-arm-linux-gnueabi
 #GNU_DIR = /usr/local/vw/zynq-2011.09
-GNU_DIR = /usr/local/vw/zynq-2016.1/gnu/arm/lin
+#GNU_DIR = /usr/local/vw/zynq-2016.1/gnu/arm/lin
 #GNU_DIR = /usr/local/Xilinx/SDK/2016.3/gnu/arm/lin
 #GNU_DIR = /APSshare/XilinxSDK/2015.4/gnu/arm/lin
 

EOF
```
<!--
##### Or just use `sed`
```bash
export _EPICS_TARGET_ARCH=arm-linux-gnueabihf
# sed -i 's/#COMMANDLINE_LIBRARY = READLINE_NCURSES/COMMANDLINE_LIBRARY = READLINE_NCURSES/' configure/os/CONFIG_SITE.Common.linux-arm
sed -i 's/GNU_TARGET = arm-xilinx-linux-gnueabi/GNU_TARGET = $(_EPICS_TARGET_ARCH)/' configure/os/CONFIG_SITE.linux-x86.linux-arm
sed -i 's/GNU_DIR = \/usr\/local\/vw\/zynq-2011.09/GNU_DIR = \/usr/' configure/os/CONFIG_SITE.linux-x86.linux-arm
# sed -i 's/#READLINE_DIR = $(GNU_DIR)/READLINE_DIR = \/usr\/$(_EPICS_TARGET_ARCH)/' configure/os/CONFIG_SITE.linux-x86.linux-arm
```
-->
##### create `CONFIG_SITE.local`
```bash
echo "CROSS_COMPILER_TARGET_ARCHS=linux-arm" > $EPICS_BASE/configure/CONFIG_SITE.local
```
#### build
```bash
make rebuild -j8
```
#### run softIoc
```bash
/epics/7.0/bin/linux-arm/softIoc
```

## cross-compile softIoc
1. first, epics base has been compiled
2. export epics base path, do distclean, then make
    ```bash
    EPICS_BASE=/epics/3.15/base && make distclean && make -j4
    ```

</details>

### konosubakonoakua · 2024-10-31T09:50:12Z

## Multiple Soft IOCs on one host (same subnet)
> [!TIP]
>  automatically create/delete an iptables rule that replaces the destination address of all incoming CA UDP traffic on each interface with the broadcast address of that interface.
> 
> Drop/link the following script into /etc/network/if-up.d/ and /etc/network/if-down.d/.
If your system does not have the ip command, consider updating it (or install package iproute2 from backports).
>
> [channel-access-reach-multiple-soft-iocs-linux](https://epics-controls.org/resources-and-support/documents/howto-documents/channel-access-reach-multiple-soft-iocs-linux/)

```bash
#!/bin/sh -e
# Called when an interface goes up / down

# Author: Ralph Lange <Ralph.Lange@gmx.de>

# Make any incoming Channel Access name resolution queries go to the broadcast address
# (to hit all IOCs on this host)

# Change this if you run CA on a non-standard port
PORT=5064

[ "$METHOD" != "none" ] || exit 0
[ "$IFACE" != "lo" ] || exit 0

line=`ip addr show $IFACE`
addr=`echo $line | grep -Po 'inet \K[\d.]+'`
bcast=`echo $line | grep -Po 'brd \K[\d.]+'`
[ -z "$addr" -o -z "$bcast" ] && return 1

if [ "$MODE" = "start" ]
then
    iptables -t nat -A PREROUTING -d $addr -p udp --dport $PORT -j DNAT --to-destination $bcast
elif [ "$MODE" = "stop" ]
then
    iptables -t nat -D PREROUTING -d $addr -p udp --dport $PORT -j DNAT --to-destination $bcast
fi

exit 0
```

> [!CAUTION]
> Note: This will not work for clients on the same host. (Adding that feature makes things a lot more complicated, and I like things to be simple.)
> 
> If you need connections between IOCs on one host, I would suggest adding the broadcast address of the loopback interface (usually 127.255.255.255) to each IOC’s EPICS_CA_ADDR_LIST setting.

```bash
export EPICS_CA_ADDR_LIST=127.255.255.255
export EPICS_PVA_ADDR_LIST=127.0.0.1
export EPICS_CA_AUTO_ADDR_LIST=no
export EPICS_PVA_AUTO_ADDR_LIST=no

```
## Multiple IOCs on the same computer but on a different subnet
> [!TIP]
> Refer to: 
> - [configure-ca](https://docs.epics-controls.org/en/latest/sys-admin/configure-ca.html)
> - [EPICS_networking](https://wiki-ext.aps.anl.gov/blc/index.php?title=EPICS_networking)


### konosubakonoakua · 2024-11-01T07:44:44Z

## References
- [epics-controls/installation-linux](https://docs.epics-controls.org/en/latest/getting-started/installation-linux.html)
- [limfx epics install manual](https://www.limfx.pro/readarticle/3088/epicsan-zhuang-yu-shi-yong-shou-ce-)
- [kira-96 EPICS IOC CROSS COMPILE](https://kira-96.github.io/posts/%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91epics%E5%92%8Cioc/)
- [StreamDevice/setup](https://paulscherrerinstitute.github.io/StreamDevice/setup.html)
- [best-practice-guidelines](https://epics-controls.org/resources-and-support/documents/howto-documents/best-practice-guidelines)
- [gnu ftp mirros](https://www.gnu.org/prep/ftp.html#asia)
- [rpi_epics](https://cmd-response.readthedocs.io/en/latest/epics/rpi_epics.html)

### konosubakonoakua · 2024-11-27T13:52:21Z


## [phoebusgen](https://github.com/als-epics/phoebusgen)
```bash
uv pip install phoebusgen
```


### konosubakonoakua · 2024-12-12T04:58:08Z

## trouble shooting
If you copy epics runtime files to SBC, errors may occur like these:
- `error while loading shared libraries: libdbRecStd.so.3.23.0: cannot open shared object file: No such file or directory`
  This is because you haven't set `LD_LIBRARY_PATH` correctly: 
  `export LD_LIBRARY_PATH=/epics/3.15/iocs/ioc-beamloss-7015/lib/linux-arm:/epics/3.15/base/lib/linux-arm:$LD_LIBRARY`
  ```
  Dynamic section at offset 0x9f00 contains 28 entries:
    Tag        Type                         Name/Value
   0x00000001 (NEEDED)                     Shared library: [libdbCore.so]
   0x00000001 (NEEDED)                     Shared library: [libCom.so]
   0x00000001 (NEEDED)                     Shared library: [libc.so.6]
   0x00000001 (NEEDED)                     Shared library: [ld-linux-armhf.so.3]
   0x0000001d (RUNPATH)                    Library runpath: [/epics/3.15/iocs/ioc-beamloss-7015/lib/linux-arm:/epics/3.15/base/lib/linux-arm]

  ```
- `symbol lookup error: ./base/bin/linux-arm/softIoc: undefined symbol: pvar_int_logClientDebug`
  This is because you haven't copy the runtime files to correct path, the symbol path is hard-coded into binaries.
- GLIBC issue
  ```
  root@beamloss7015:~# /epics/3.15/iocs/ioc-beamloss-7015/bin/linux-arm/beamloss
  /epics/3.15/iocs/ioc-beamloss-7015/bin/linux-arm/beamloss: /lib/libc.so.6: version `GLIBC_2.34' not found (required by /epics/3.15/iocs/ioc-beamloss-7015/bin/linux-arm/beamloss)
  /epics/3.15/iocs/ioc-beamloss-7015/bin/linux-arm/beamloss: /lib/libc.so.6: version `GLIBC_2.34' not found (required by /epics/3.15/iocs/ioc-beamloss-7015/lib/linux-arm/libbeamlossSupport.so)
  ```
  This is because the cross-compiler use glibc 2.34 while the target only have glibc 2.28 (petalinux 2019.1), and higher version cannot run on lower one.
  Use this cmd to check glibc version of executables.
  ```bash
  nm bin/linux-arm/softIoc | grep GLIBC_
  ```
  Got:
  ```
  base on  HEAD (3be67ac) [$!+] took 32s                                                                                                                               
  ❯ nm bin/linux-arm/softIoc | grep GLIBC_                                                                                                                              
           U abort@@GLIBC_2.4                                                                                                                                           
           U getenv@@GLIBC_2.4                                                                                                                                          
           U __libc_start_main@@GLIBC_2.34                                                                                                                               
           U printf@@GLIBC_2.4                                                                                                                                          
           U puts@@GLIBC_2.4                                                                                                                                            
           U strcmp@@GLIBC_2.4                                                                                                                                          
           U strrchr@@GLIBC_2.4 
  ```
  You need to use cross-compiler provided by Xilinx SDK:
  ```
  GNU_DIR = /tools/Xilinx/SDK/2019.1/gnu/aarch32/lin/gcc-arm-linux-gnueabi
  ```

### konosubakonoakua · 2025-01-23T03:56:52Z

## Prebuilt
Refer to #136 

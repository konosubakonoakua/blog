---
number: 169
title: "[epics] epics issues"
state: open
url: https://github.com/konosubakonoakua/blog/issues/169
author: konosubakonoakua
createdAt: 2025-04-08T10:35:50Z
updatedAt: 2025-07-11T08:06:07Z
labels: []
---

# 169 [epics] epics issues

### first thing first
read manuals!!!!!
- https://docs.epics-controls.org/en/latest/getting-started/EPICS_Intro.html
- https://epics.anl.gov/base/R7-0/9.php
  - https://docs.epics-controls.org/en/latest/internal/ca_protocol.html
  - https://epics.anl.gov/base/R7-0/9-docs/CAref.html
- https://epics.anl.gov/base/R3-15/9.php
  - https://epics.anl.gov/base/R3-15/6-docs/AppDevGuide/AppDevGuide.html
  - https://epics.anl.gov/base/R3-15/9-docs/CAref.html
  - https://epics.anl.gov/base/R3-15/6-docs/CAproto/index.html



---

## Comments

### konosubakonoakua · 2025-04-09T06:33:38Z

### `recGblRecordError: ai: init_record Error (514,5) PV: BLM:ZynqMP:PS:Temp`
- `#include <epicsExport.h>`
- wrong ai dset
  ```c
  aidset devAiSoft = {
      { 6, NULL, NULL, init_record, NULL },
      read_ai, NULL
  };
  epicsExportAddress(dset, devAiSoft);
  ```
### `recGblRecordError: wf: init_record Error (514,5) PV: WF:CH0`
- `read_wf` must be implemented.

### konosubakonoakua · 2025-04-09T09:30:12Z

### `iocInit` in `st.cmd`, but not working
- try to add a blank line below iocInit

### konosubakonoakua · 2025-04-09T11:12:05Z

### record driver working, but caget pv = 0
- use `prec->rval=val` instead of `prec->val=val`, later is EGU value，then implement linear_conv
- `read_ai()` return `2`, if already obtained EGU value

### konosubakonoakua · 2025-04-10T10:44:22Z

### IOC build or instatllation error
```text
IOC build or installation error:
    The xxxRecord defined in the DBD file has 68 fields,
    but the record support code was built with 454.
```
- `make clean install && make`

### konosubakonoakua · 2025-04-21T02:17:40Z

### wired intersected zeros & values of waveform record
```log
BLM:WAV_AXIS_2000 2025-04-21 10:06:04.064604 2000 0 0 1 0 2 0 3 0 4 0 5 0 6 0 7 0 8 0 9 0
```
You should check whether the type of your C array really matched with EPICS TYPES (ULONG something).
On an aarch64 system, `ULONG` responds to `uint32_t`, not `unsigned long` which is 64bit

### konosubakonoakua · 2025-04-22T01:39:13Z

### epics multiple waveform pv not aligned on opi but aligned if dumped to file
- maybe the update frequency is too fast

### konosubakonoakua · 2025-04-25T09:19:31Z

### Logic Error: process Target x ...
- error
  ```text
  epics> Logic Error: processTarget 1 from 0xaaaad34b8d10, BLM(0xaaaad32db430) -> BLM:WAV_CH2_INTEGRAL_REALTIME_10US(0xaaaad32fe8b0)
  Logic Error: processTarget 2 from 0xaaaad34b8d10, BLM(0xaaaad32db430) -> BLM:WAV_CH2_INTEGRAL_REALTIME_10US(0xaaaad32fe8b0)
  Logic Error: processTarget 1 from 0xaaaad34b8d10, BLM(0xaaaad32db430) -> BLM:WAV_CH4_INTEGRAL_REALTIME_10US(0xaaaad32ff1f0)
  ```
- root cause
  dbputfiled outlinks must protected with dbscanlock
- use `dbscanlock` & `dbscanunlock`

### konosubakonoakua · 2025-06-19T15:56:19Z

### debug mode
- add to `CONFIG_SITE`
  ```makefile
  # Debug
  USR_CFLAGS += -g -O0
  USR_CXXFLAGS += -g -O0
  ```

### konosubakonoakua · 2025-06-25T03:06:30Z

### autosave not working
- check `auto_settings.req`
- call `create_moniter_set` before `iocInit`
  ```bash
  #- Autosave config
  #- https://epics-modules.github.io/autosave/autoSaveRestore.html
  set_requestfile_path("${TOP}/iocBoot/${IOC}", "")
  set_requestfile_path("${TOP}/iocBoot/${IOC}", "autosave")
  set_savefile_path("${AUTOSAVES}")
  set_pass0_restoreFile("auto_settings.sav", "P=${P}")
  set_pass1_restoreFile("auto_settings.sav", "P=${P}")
  #- set_restoreSet_numSeqFiles(3)
  #- set_restoreSet_SeqPeriodInSeconds(600)
  #- set_restoreSet_RetrySeconds(60)
  #- set_restoreSet_CAReconnect(1)
  #- set_restoreSet_CallbackTimeout(-1)
  #- create_monitor_set(<name>.req, <time in secs>, <macro>)
  create_monitor_set("auto_settings.req", 1, "P=${P}")
  
  cd "${TOP}/iocBoot/${IOC}"
  iocInit
  ```
### autosave not restoring
- call `set_pass0_restoreFile`

### konosubakonoakua · 2025-07-11T08:06:07Z


### `caget` timed out, but `dbl/dbgf` works
try:
```bash
export EPICS_CA_ADDR_LIST="localhost"
export EPICS_CA_AUTO_ADDR_LIST=NO
export EPICS_CA_MAX_ARRAY_BYTES=40000000
```


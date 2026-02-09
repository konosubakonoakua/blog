---
number: 160
title: "[epics] epics logging system"
state: open
url: https://github.com/konosubakonoakua/blog/issues/160
author: konosubakonoakua
createdAt: 2025-03-03T09:17:41Z
updatedAt: 2025-03-03T09:17:49Z
labels: []
---

# 160 [epics] epics logging system

## EPICS IOC LOGGING
- [ioclogserver](https://docs.epics-controls.org/en/latest/appdevguide/IOCErrorLogging.html#ioclogserver)
- [CONFIG_SITE_ENV#L78](https://github.com/epics-base/epics-base/blob/131578124b2e5baf7cb1d6b8fd3e2a7b75001fe7/configure/CONFIG_SITE_ENV#L78)
  ```bash
  # Log Server:
  # EPICS_IOC_LOG_INET 
  #	Log server ip addr.
  # EPICS_IOC_LOG_FILE_NAME 
  #	pathname to the log file.
  # EPICS_IOC_LOG_FILE_LIMIT 
  #	maximum log file size.
  # EPICS_IOC_LOG_FILE_COMMAND 
  #	A shell command string used to obtain a new 
  #       path name in response to SIGHUP - the new path name will
  #       replace any path name supplied in EPICS_IOC_LOG_FILE_NAME
  ```
```bash
export EPICS_IOC_LOG_FILE_NAME=loooooog.log
export EPICS_IOC_LOG_FILE_LIMIT=1000000
export EPICS_IOC_LOG_PORT=12345
```
- [initialize-logging](https://docs.epics-controls.org/en/latest/internal/IOCInit.html#initialize-logging)
```bash
# st.cmd
epicsEnvSet("EPICS_IOC_LOG_PORT", "<port>")
epicsEnvSet("EPICS_IOC_LOG_INET", "<inet addr>") # use localhost if only local
iocLogInit
```

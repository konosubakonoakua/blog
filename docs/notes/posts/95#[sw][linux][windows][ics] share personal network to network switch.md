---
number: 95
title: "[sw][linux][windows][ics] share personal network to network switch"
state: open
url: https://github.com/konosubakonoakua/blog/issues/95
author: konosubakonoakua
createdAt: 2024-09-23T14:31:53Z
updatedAt: 2025-08-18T05:26:46Z
labels: ["Linux"]
---

# 95 [sw][linux][windows][ics] share personal network to network switch

## computer A with working network (windows)
> [!IMPORTANT]
> If you have windows (host) rebooted, you need to restart the share process, disable then enable.
> Or make it persistant. [Follow tutorial here.](https://woshub.com/internet-connection-sharing-not-working-windows-reboot/)
> ```powershell
> New-ItemProperty -Path HKLM:\Software\Microsoft\Windows\CurrentVersion\SharedAccess -Name EnableRebootPersistConnection -Value 1 -PropertyType dword
> Set-Service SharedAccess –startuptype automatic –passthru
> Start-Service SharedAccess
> ```
- adapter 1 connect to outside network (to be shared)
    <img src=https://github.com/user-attachments/assets/cb4b4387-6335-40c9-a4aa-94c381da9e07 width="200" />

- adapter 2 connect to switch (any port) (shared via it)
    <img src=https://github.com/user-attachments/assets/66dc1deb-7be6-4b58-b964-d0ae96aee5bc style="width:auto; height:50%; "/>

- don't forget to set all dns servers to `114.114.114.114` or `192.168.137.1`

- disable auto powersave
  <img width="398" height="187" alt="Image" src="https://github.com/user-attachments/assets/11d3faea-4544-46a7-a608-ccd7df015e7f" />

---

## Comments

### konosubakonoakua · 2024-09-23T14:37:48Z

## computer B connected to net switch (ubuntu 22)
```bash
#!/bin/bash

export NIC=eno1
export GATEWAY=192.168.137.1
export IP0=192.168.137.2
export IP1=10.9.8.8
export IP2=10.10.9.235

export CONNECTION=$(nmcli -t -f DEVICE,CONNECTION device status | grep $NIC | awk -F: '{print $2}')

sudo nmcli connection modify "$CONNECTION" ipv4.addresses $IP0/24
sudo nmcli connection modify "$CONNECTION" ipv4.gateway $GATEWAY
sudo nmcli connection modify "$CONNECTION" ipv4.dns $GATEWAY
sudo nmcli connection modify "$CONNECTION" ipv4.method manual
sudo nmcli connection modify "$CONNECTION" +ipv4.addresses $IP1/24
sudo nmcli connection modify "$CONNECTION" +ipv4.addresses $IP2/24

sudo nmcli connection up "$CONNECTION"

ip route show

```
then we get
<img src="https://github.com/user-attachments/assets/ad36cb9e-6f6c-40ed-b1b0-3c4b45a813bf" width="400" />



### konosubakonoakua · 2024-09-23T16:27:15Z

https://ubuntu.com/core/docs/networkmanager/edit-connections

### konosubakonoakua · 2024-09-27T09:28:05Z

## Troubleshooting
- Cannot access network but can be reached by ping (DHCP working, but IP forwarding not)
  - try re-enable the share
  - disable usb powersave
  - enable ics autostart
  - enable sharedaccess reboot persist
- https://www.wqy.ac.cn/p/2410-ics-persist/
  ```powershell
  # fix Windows 10 ICS not working after reboot
  # https://support.microsoft.com/en-us/help/4055559/ics-doesn-t-work-after-computer-or-service-restart-on-windows-10
  [microsoft.win32.registry]::SetValue("HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\SharedAccess", "EnableRebootPersistConnection", 0x01)
  
  # Enable IP Forwarding
  # https://serverfault.com/questions/929081/how-can-i-enable-packet-forwarding-on-windows/929089#929089
  Set-NetIPInterface -Forwarding Enabled
  # Check IP Forwarding Status
  Get-NetIPInterface | Select-Object ifIndex,InterfaceAlias,AddressFamily,ConnectionState,Forwarding | Sort-Object -Property IfIndex | Format-Table
  
  # Change the Internet Connection Sharing (ICS) service Startup type to Automatic.
  Set-Service SharedAccess -StartupType Automatic
  # Start ICS service for now
  Start-Service SharedAccess
  # Check ICS service Status
  Get-Service SharedAccess
  ```



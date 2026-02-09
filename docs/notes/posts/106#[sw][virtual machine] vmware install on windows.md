---
number: 106
title: "[sw][virtual machine] vmware install on windows"
state: open
url: https://github.com/konosubakonoakua/blog/issues/106
author: konosubakonoakua
createdAt: 2024-10-01T12:59:43Z
updatedAt: 2026-01-17T16:27:41Z
labels: []
---

# 106 [sw][virtual machine] vmware install on windows

## download link (need broadcom account)
- https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Workstation%20Pro&freeDownloads=true
## another download link (without vm tool)
- https://softwareupdate.vmware.com/cds/vmw-desktop/ws/
## get vm-tools automatically
<details>

<img src="https://github.com/user-attachments/assets/250d6b47-0d3a-465b-a8e0-54ee30f60103" width="50%" />
</details>

---

## Comments

### konosubakonoakua · 2024-10-04T15:33:48Z

## ubuntu22 freezes or has black screen after inactivity
> [!CAUTION]
> Don't use vmware 17.6.0 version with linux kernel 6.8
> Freeze confirmed also on 17.5.2

### broadcom community threads
  - [vm-freezes-after-inactivity](https://community.broadcom.com/vmware-cloud-foundation/question/vm-freezes-after-inactivity)
  - [solution (maybe)](https://community.broadcom.com/vmware-cloud-foundation/question/problem-with-176-update-on-ubuntu-2204#ViewAnswer_779eaac6-c0d4-47e6-b409-0191c5edc93f-answer-container)
### Try to disable 3d acceleration. (IDK whether it works)

### konosubakonoakua · 2024-10-04T18:27:44Z

## vmware shared folder on windows
- [offcial guide](https://docs.vmware.com/en/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-AB5C80FE-9B8A-4899-8186-3DB8201B1758.html)
- Setting->shared folder->new->properties->fill name&path->ok
  <details>

  ![image](../assets/106/1.png)

  </details>

- Linux Side:
  ```bash
  sudo mkdir -p /mnt/winshared
  sudo chown rf:rf /mnt/winshared
  /usr/bin/vmhgfs-fuse .host:/ /mnt/winshared -o subtype=vmhgfs-fuse
  ```
  - In case that you want to use systemd: https://github.com/konosubakonoakua/blog/issues/117#issue-2607557649

### konosubakonoakua · 2024-10-04T20:46:02Z

## Disk Optimization
- Fragment cleanup & Compact (on host, not helpful)
  - both need extras disk space on host, be careful
  <details>
  
  <img src="https://github.com/user-attachments/assets/2b8b01b3-c4d4-42dc-8f2e-07783d9770f8" width="60%" />
  </details>
- disk shrink (on guest, helpful)
  ```bash
  sudo vmware-toolbox-cmd disk shrink /
  ```
  <details>
  
  ![image](../assets/106/2.png)
  </details>

- References
  - https://www.cnblogs.com/fensnote/p/13436463.html

### konosubakonoakua · 2024-10-05T12:41:49Z

## clipboard not synced when connected via RDP
https://github.com/vmware/open-vm-tools/issues/740

### konosubakonoakua · 2024-10-08T08:32:24Z

## Turn off Hyper-V if getting sluggish
[virtualization-apps-not-work-with-hyper-v](https://learn.microsoft.com/en-us/troubleshoot/windows-client/application-management/virtualization-apps-not-work-with-hyper-v)
```cmd
bcdedit /set hypervisorlaunchtype off
```
or
```powershell
Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-Hypervisor
```
then reboot.

### konosubakonoakua · 2024-10-25T08:23:15Z

## VMware Workstation cannot connect to the virtual machine
> [!CAUTION]
> VMware Workstation cannot connect to the virtual
> machine. Make sure you have rights to run the
> program, access all directories the program uses,
> and access all directories for temporary files.
### The system cannot find the file specified.
> [!CAUTION]
> Failed to connect pipe to virtual machine: The
> system cannot find the file specified.

#### Solutions
- run vmware as admin
- install common vcredist
  ```powershell
  winget install Microsoft.VCRedist.2015+.x86
  winget install Microsoft.VCRedist.2015+.x64
  ```
### The VMX process exited prematurely
> [!CAUTION] 
> The VMX process exited prematurely
#### Solutions
- run `vmware-vmx.exe` manually to find out the real error
- missing `RESAMPLEDMO.DLL`, download them here: [resampledmo.dll.zip](https://github.com/user-attachments/files/17528592/resampledmo.dll.zip)

### konosubakonoakua · 2024-10-31T02:56:34Z

## optimization for performance
### Fit all virtual machine memory into reserved host RAM
- `Edit`->`Preferences`->`Memory`->`Fit all virtual machine memory into reserved host RAM`
    <details>
    
    <img src="https://github.com/user-attachments/assets/aa394145-fb00-4f91-95d0-d1459c637641" wdith="70%"/>
    
    </details>
### Disable memory page trimming & Input grabbed

- `Virtual Machine Settings`->`Options`->`Advanced`->`Process priorites`->`Input grabbed:`->`High`

- `Virtual Machine Settings`->`Options`->`Advanced`->`Process priorites`->`Input ungrabbed:`->`Low`

- `Virtual Machine Settings`->`Options`->`Advanced`->`Settings`->`Dsiable memory page trimming`

    <details>
    
    <img src="https://github.com/user-attachments/assets/8ae3ed6f-1cd2-4008-a6a4-422c0fb36d17" wdith="70%"/>

   
    </details>

### konosubakonoakua · 2024-10-31T03:01:27Z

## Vmware Networking
### Bridged mode (vm has its own IP)
1. `Edit`->`Virtual Network Editor`
    <details>
    
    <img src="https://github.com/user-attachments/assets/6152ec34-3b52-4167-b9c4-94888464978c" wdith="70%"/>
    
    </details>

2. `VMnet Information`->`Bridged(connect VMs directly to the external network)`->`Bridged to:`->`Intel(R) Ethernet Controller (3) I225-V`
    <details>
    
    <img src="https://github.com/user-attachments/assets/dbfedfc0-9f6f-4ea6-a526-5fd39a15d69e" wdith="70%"/>
    
    </details>

### mdns not working

<details>

- check if vm select `bridged mode`
  ![Image](../assets/106/3.png)
  
</details>


### konosubakonoakua · 2025-05-14T10:52:17Z

## damaged file system restore (e.g. powerloss)

<details>

![Image](../assets/106/4.png)

</details>

- fsck
  ```bash
  fsck -y /dev/sda1
  ```

### konosubakonoakua · 2026-01-17T14:53:39Z

## Virtualbox export to VMware

- In Host OS
  ```bash
  VBoxManage clonehd "PYNQ3.1.vdi" "PYNQ3_1.vmdk" --format VMDK
  ```
- In guest OS
  ```bash
  sudo systemctl disable vboxadd-service
  sudo systemctl disable vboxadd
  sudo /opt/VBoxGuestAdditions-*/uninstall.sh
  ```

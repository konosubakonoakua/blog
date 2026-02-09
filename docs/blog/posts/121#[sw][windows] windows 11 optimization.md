---
number: 121
title: "[sw][windows] windows 11 optimization"
state: open
url: https://github.com/konosubakonoakua/blog/issues/121
author: konosubakonoakua
createdAt: 2024-10-31T03:08:08Z
updatedAt: 2025-05-14T11:34:00Z
labels: []
---

# 121 [sw][windows] windows 11 optimization

## Disable auto-restart when updates are installed
### Dism++
<details>

<summary>
screenshots
</summary>

![dism++](../assets/121/1.png)
</details>

### Local Group Policy Editor

<details>

<summary>
screenshots
</summary>

![Local Group Policy Editor](../assets/121/2.png)

![No auto-restart with logged on users for scheduled automatic updates installations](../assets/121/3.png)

![configure automatic updates](../assets/121/4.png)

</details>

---

## Comments

### konosubakonoakua · 2025-05-14T11:33:59Z

## restore health
```powershell
dism /online /Cleanup-image /Restorehealth
```

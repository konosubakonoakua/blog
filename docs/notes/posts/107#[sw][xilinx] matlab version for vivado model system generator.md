---
number: 107
title: "[sw][xilinx] matlab version for vivado model system generator"
state: open
url: https://github.com/konosubakonoakua/blog/issues/107
author: konosubakonoakua
createdAt: 2024-10-03T04:33:28Z
updatedAt: 2025-05-29T07:28:23Z
labels: ["FPGA", "ZYNQ", "matlab"]
---

# 107 [sw][xilinx] matlab version for vivado model system generator

## matlab for HDL
![image](../assets/107/1.png)

## vivado 2019.1
> [!IMPORTANT]
> `R2016a/b` or `R2017a/b` supported for vivado 2019.1

![image](../assets/107/2.png)

## vivado 2024.1
> [!IMPORTANT]
> `R2022a(Update 6 or later)`, `R2022b(Update 6 or later)`, `R2023a`, `R2023b` for vivado 2024.1

![image](../assets/107/3.png)


---

## Comments

### konosubakonoakua · 2024-10-04T04:34:21Z

## Troubleshoot
### An operation is in progress when quit matlab
The matlab is searching for license, pass its path via command line in the desktop shortcut.
```bash
"C:\Program Files\MATLAB\R2018a\bin\matlab.exe" -c "C:\Program Files\MATLAB\R2018a\licenses\license_standalone.lic"
```

### konosubakonoakua · 2025-05-29T07:28:21Z

- [Which versions of Vivado are supported with which release of HDL Coder?](https://ww2.mathworks.cn/matlabcentral/answers/518421-which-versions-of-vivado-are-supported-with-which-release-of-hdl-coder)
- [添加版本matlab支持](https://blog.csdn.net/qq_45910789/article/details/144537901)

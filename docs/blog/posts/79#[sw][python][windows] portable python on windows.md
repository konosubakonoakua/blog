---
number: 79
title: "[sw][python][windows] portable python on windows"
state: open
url: https://github.com/konosubakonoakua/blog/issues/79
author: konosubakonoakua
createdAt: 2024-08-28T02:58:08Z
updatedAt: 2024-11-27T02:01:19Z
labels: ["Question", "python"]
---

# 79 [sw][python][windows] portable python on windows

## portable
https://github.com/konosubakonoakua/python-embed/releases/tag/py311-with-pip

坑得很，
关于怎么安装 pip
先 python get-pip.py
然后用 pip_bodge install --force-reinstall pip 覆盖原先安装的 pip
然后以后安装都用 pip_bodge install ...
否则不能实现 portable，因为 智障 pip 会把路径硬编码进二进制exe

```python
# pip_bodge.cmd
@echo off
python -c "from pip._internal.cli.main import main;import sys;sys.executable='python.exe';main()" %*
```

---

## Comments

### konosubakonoakua · 2024-11-27T02:00:21Z

## jupyter-lab 启动
```cmd
@echo off
set VENV_PATH=.\venv
call %VENV_PATH%\Scripts\activate
if %ERRORLEVEL% NEQ 0 (
    echo Failed to activate %VENV_PATH%
    exit /b 1
)
jupyter-lab
call %VENV_PATH%\Scripts\deactivate
```

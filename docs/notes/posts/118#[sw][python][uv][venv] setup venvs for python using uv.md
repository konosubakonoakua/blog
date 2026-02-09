---
number: 118
title: "[sw][python][uv][venv] setup venvs for python using uv"
state: open
url: https://github.com/konosubakonoakua/blog/issues/118
author: konosubakonoakua
createdAt: 2024-10-23T07:50:53Z
updatedAt: 2024-11-12T12:35:39Z
labels: []
---

# 118 [sw][python][uv][venv] setup venvs for python using uv

## windows

Install scoop
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

Install uv
```powershell
scoop install uv
```

In case that you don't want to use scoop:
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
Or
```powershell
winget install --id=astral-sh.uv -e
```

Auto-completion
```powershell
Add-Content -Path $PROFILE -Value '(& uv generate-shell-completion powershell) | Out-String | Invoke-Expression'
Add-Content -Path $PROFILE -Value '(& uvx --generate-shell-completion powershell) | Out-String | Invoke-Expression'
. $PROFILE
```

Mirror
[Config files](https://docs.astral.sh/uv/configuration/files/#configuration-files)
```powershell
New-Item -Path "$env:APPDATA\uv" -ItemType Directory -Force; Set-Content -Path "$env:APPDATA\uv\uv.toml" -Value @"
index-url = "https://mirrors.bfsu.edu.cn/pypi/web/simple"
"@
```

Make dir
```powershell
mkdir $env:USERPROFILE\.virtualenvs
cd $env:USERPROFILE\.virtualenvs
```

Create venv
```powershell
uv venv --seed -p 3.12 -n instr --preview
```

Activate
```powershell
. $env:USERPROFILE\.virtualenvs\instr\Scripts\activate.ps1
```


---

## Comments

### konosubakonoakua · 2024-10-23T08:24:03Z

## linux
Install uv
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
SHELL autocompletion
```bash
echo 'eval "$(uv --generate-shell-completion bash)"' >> ~/.bashrc
echo 'eval "$(uvx --generate-shell-completion bash)"' >> ~/.bashrc
source ~/.bashrc
```
Mirror
[Config files](https://docs.astral.sh/uv/configuration/files/#configuration-files)
```bash
cat > ~/.config/uv/uv.toml << 'EOF'
index-url = "https://mirrors.bfsu.edu.cn/pypi/web/simple"
EOF
```
Make dir
```bash
mkdir ~/.virtualenvs
cd .virtualenvs
```
Create venv
```bash
uv venv --seed -p 3.12 -n instr --preview
```
Activate
```bash
source ~/.virtualenvs/instr/bin/activate
```

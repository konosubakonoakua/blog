---
number: 123
title: "[sw][linux] ssh config"
state: open
url: https://github.com/konosubakonoakua/blog/issues/123
author: konosubakonoakua
createdAt: 2024-11-26T03:24:12Z
updatedAt: 2025-04-02T02:17:59Z
labels: []
---

# 123 [sw][linux] ssh config

## keep alive
On Client Side:
linux
```bash
cat >> ~/.ssh/config << 'EOF'
Host *
    TCPKeepAlive yes
    ServerAliveInterval 60
    ServerAliveCountMax 3
EOF
```
windows
```powershell
$configContent = @"

Host *
    TCPKeepAlive yes
    ServerAliveInterval 60
    ServerAliveCountMax 3
"@
Add-Content -Path "$env:USERPROFILE\.ssh\config" -Value $configContent
```
Or on Server Side:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo sed -i \
  -e '/^#TCPKeepAlive/c\TCPKeepAlive yes' \
  -e '/^#ClientAliveInterval/c\ClientAliveInterval 60' \
  -e '/^#ClientAliveCountMax/c\ClientAliveCountMax 3' \
  /etc/ssh/sshd_config
```

---

## Comments

### konosubakonoakua · 2024-12-12T01:18:26Z

## Trouble shoot
> Unable to negotiate with 192.168.137.222 port 22: no matching host key type found. Their offer: ssh-rsa

Solution:
```bash
ssh -oHostKeyAlgorithms=+ssh-rsa root@192.168.137.222
```

### konosubakonoakua · 2025-04-02T02:17:58Z

## allow root
```bash
sudo sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
sudo sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
```

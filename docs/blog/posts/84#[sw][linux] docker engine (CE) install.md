---
number: 84
title: "[sw][linux] docker engine (CE) install"
state: open
url: https://github.com/konosubakonoakua/blog/issues/84
author: konosubakonoakua
createdAt: 2024-09-21T18:31:17Z
updatedAt: 2025-03-31T03:01:57Z
labels: ["Linux"]
---

# 84 [sw][linux] docker engine (CE) install

## guides
[official guide](https://docs.docker.com/engine/install/ubuntu)
### remove unofficial packages
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
### setup apt
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
```
```bash
case $(uname -a) in
  *buntu*) HOST_OS=ubuntu;;
  *ebian*) HOST_OS=debian;;
  *)
    echo "Unsupported system"
    ;;
esac
```

```bash
# ubuntu18
# Add Docker's official GPG key:
curl -fsSL https://download.docker.com/linux/$HOST_OS/gpg | sudo apt-key add -
```
Or
```bash
# ubuntu 22
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/$HOST_OS/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```bash
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/${HOST_OS} \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install
```bash
sudo apt-get update
```
### install docker packages
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## Comments

### konosubakonoakua · 2024-09-21T20:16:11Z

## docker mirrors
https://github.com/konosubakonoakua/blog/issues/86#issuecomment-2365307898
- https://jockerhub.com/
- https://docker.1panel.dev/
```bash
sudo tee /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": ["https://hub.1panel.dev"]
}
EOF
sudo systemctl daemon-reload && sudo systemctl restart docker
```

### konosubakonoakua · 2024-11-26T02:29:16Z

### Verify
```bash
sudo docker run hello-world
```
If failed to pull image, follow #86.
### postinstall
#### [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
- Check if you already have a Docker group:
    ```bash
    groups
    ```
- If not
    ```bash
    # sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker
    ```
- If you have `$HOME/.docker:
    ```bash
    sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
    sudo chmod g+rwx "$HOME/.docker" -R
    ```
- Try
    ```bash
    docker run hello-world
    ```

### trouble shooting
- [rootless](https://docs.docker.com/engine/security/rootless/)
- [firewall](https://docs.docker.com/engine/network/packet-filtering-firewalls/#docker-and-ufw)


### konosubakonoakua · 2025-03-31T01:57:35Z

## proxy
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf << EOF
[Service]                                                                                                                                                              
Environment="HTTP_PROXY=$http_proxy"                                                                                                                       
Environment="HTTPS_PROXY=$https_proxy"                                                                                                                      
Environment="NO_PROXY=localhost,127.0.0.1,"
EOF
```
```bash
sudo mv /etc/systemd/system/docker.service.d/http-proxy.conf /etc/systemd/system/docker.service.d/http-proxy.conf.disabled
```
```bash
sudo mv /etc/systemd/system/docker.service.d/http-proxy.conf.disabled /etc/systemd/system/docker.service.d/http-proxy.conf
```
```bash
sudo systemctl daemon-reload && sudo systemctl restart docker && systemctl show --property=Environment docker
```

### konosubakonoakua · 2025-03-31T02:42:57Z

## References
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04

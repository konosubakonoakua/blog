---
number: 166
title: "[linux] ubuntu 18 setup"
state: open
url: https://github.com/konosubakonoakua/blog/issues/166
author: konosubakonoakua
createdAt: 2025-03-25T03:33:06Z
updatedAt: 2025-03-28T06:44:55Z
labels: []
---

# 166 [linux] ubuntu 18 setup

## system settings
### disable animations (sluggish if disabled, I choose to disable 3d accel)
```bash
gsettings set org.gnome.desktop.interface enable-animations false
```
### fix `System Program Problem Detected`
```bash
sudo rm /var/crash/*
sudo systemctl disable apport
sudo apt purge apport
```

---

## Comments

### konosubakonoakua · 2025-03-25T08:50:14Z

## bashrc

<details>

```bash
## history
HISTSIZE=-1
HISTFILESIZE=-1
HISTCONTROL=ignoreboth
PROMPT_COMMAND="history -a; $PROMPT_COMMAND"

## proxy
export RF_PROXY_IP="192.168.2.3"
export RF_PROXY_PORT="7899"
proxy_set() {
    local proxy_address="http://$RF_PROXY_IP:$RF_PROXY_PORT"
    local current_http_proxy="$http_proxy"
    local current_https_proxy="$https_proxy"
    export http_proxy="$proxy_address"
    export https_proxy="$proxy_address"
    echo "Proxy environment variables set."
}
proxy_clear() {
    unset http_proxy
    unset https_proxy
    echo "Proxy environment variables cleared."
}
proxy_toggle() {
    local proxy_address="http://$RF_PROXY_IP:$RF_PROXY_PORT"
    local current_http_proxy="$http_proxy"
    local current_https_proxy="$https_proxy"

    if [[ "$current_http_proxy" == "$proxy_address" && "$current_https_proxy" == "$proxy_address" ]]; then
        unset http_proxy
        unset https_proxy
        echo "Proxy environment variables cleared."
    else
        export http_proxy="$proxy_address"
        export https_proxy="$proxy_address"
        echo "Proxy environment variables set."
    fi
}

proxy_set

## tools
alias fvim='NVIM_APPNAME="fei.nvim" nvim' # will save plugins at ~/.local/share/fei.nvim
export PATH="$HOME/.local/bin:$PATH"
source "$HOME/.cargo/env"
[ -f ~/.fzf.bash ] && source ~/.fzf.bash

FNM_PATH="$HOME/.local/share/fnm"
if [ -d "$FNM_PATH" ]; then
  export PATH="$FNM_PATH:$PATH"
  eval "`fnm env`"
fi

eval "$(zoxide init bash)"
eval "$(uv --generate-shell-completion bash)"
eval "$(uvx --generate-shell-completion bash)"

## vivado & petalinux
export VIVADO_PATH="/tools/Xilinx/Vivado/2021.1"
export VIVADO_SETTINGS="$VIVADO_PATH/settings64.sh"
export PETALINUX_PATH="/tools/Xilinx/PetaLinux/2021.1/"
export PETALINUX_SETTINGS="$PETALINUX_PATH/settings.sh"
alias vivado2021='source $VIVADO_SETTINGS'
alias petalinux2021='source $PETALINUX_SETTINGS'


```
</details>

### konosubakonoakua · 2025-03-25T09:04:59Z

## input method (sogou pinyin)

> [!CAUTION]
> If cannot display chinese characters, then try to move down or up sogou pinyin in input method list.

<details>

- Add language support for chinese
  ![Image](../assets/166/1.png)
  
- Install deps
  ```bash
  sudo apt-get purge sogoupinyin
  sudo apt -y --purge remove fcitx
  sudo apt clean fcitx
  ```
  ```bash
  sudo apt -y install fcitx fcitx-bin fcitx-table fcitx-table-all fcitx-config-gtk fcitx-libs libfcitx-qt0 libopencc2 libopencc2-data libqt4-opengl libqtwebkit4
  ```
- [Install sogou pinyin](https://shurufa.sogou.com/linux)
  ```bash
  wget http://cdn2.ime.sogou.com/dl/index/1571302197/sogoupinyin_2.3.1.0112_amd64.deb
  sudo dpkg -i sogoupinyin_2.3.1.0112_amd64.deb
  ```
- Config input method
  ![Image](../assets/166/2.png)


</details>

### konosubakonoakua · 2025-03-28T04:39:43Z

## tools

<details>

```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git git-lfs
git config --global init.defaultBranch main
git config --global user.email "ailike_meow@qq.com"
git config --global user.name konosubakonoakua
ssh-keygen
# git-lfs install
```
```bash
git clone https://github.com/konosubakonoakua/.dotfiles.git
cd ~/.dotfiles/scripts/install
mkdir -p ~/.local/bin
./install_ripgrep.sh
./install_zoixde.sh
./install_zig.sh
./install_fzf.sh
./install_zellij.sh
./install_tree-sitter.sh
./install_nerdfonts.sh
./install_lazygit.sh
./install_githubcli.sh
./install_btop.sh
./install_jq_stuffs.sh
```
```bash
sudo apt install -y vim xsel python3-pip python3-venv
```
```bash
sudo snap refresh
sudo snap install nvim --classic
sudo snap install glow
```
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install fd-find
```

</details>

### konosubakonoakua · 2025-03-28T04:50:21Z

## neovim
```bash
git clone git@github.com:konosubakonoakua/fei.nvim.git ~/.config/fei.nvim
tee -a ~/.bashrc << EOF
alias fvim='NVIM_APPNAME="fei.nvim" nvim' # will save plugins at ~/.local/share/fei.nvim
EOF
```

### konosubakonoakua · 2025-03-28T06:05:32Z

## python
### uv
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

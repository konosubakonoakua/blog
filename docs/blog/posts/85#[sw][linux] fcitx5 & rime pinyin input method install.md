---
number: 85
title: "[sw][linux] fcitx5 & rime pinyin input method install"
state: open
url: https://github.com/konosubakonoakua/blog/issues/85
author: konosubakonoakua
createdAt: 2024-09-21T19:25:22Z
updatedAt: 2024-10-17T08:53:41Z
labels: ["Linux", "rime"]
---

# 85 [sw][linux] fcitx5 & rime pinyin input method install

## fcitx5
```bash
sudo apt install fcitx5
#im-config
```
config `~/.xprofile`
```bash
tee -a ~/.xprofile << EOF
export GTK_IM_MODULE=fcitx5
export QT_IM_MODULE=fcitx5
export XMODIFIERS=\"@im=fcitx5\"
export LANG=\"zh_CN.UTF-8\"
export LC_CTYPE=\"zh_CN.UTF-8\"
EOF
```
## rime
- install
    ```bash
    sudo apt install fcitx5-rime
    ```
- clone config from github
    ```bash
    mkdir -p ~/.local/share/fcitx5/
    cd ~/.local/share/fcitx5 \
      && git clone --depth=1 https://github.com/iDvel/rime-ice \
      && ln -s "$(pwd)/rime-ice" "$(pwd)/rime"
    ```

- custom
    - explain
        ![image](../assets/85/1.png)
    - diff
        ```diff
        git apply << EOF
        diff --git a/default.yaml b/default.yaml
        index f3d6f73..4ac4cc9 100644
        --- a/default.yaml
        +++ b/default.yaml
        @@ -10,19 +10,19 @@ config_version: '2024-02-05'
         schema_list:
           # 可以直接删除或注释不需要的方案，对应的 *.schema.yaml 方案文件也可以直接删除
           # 除了 t9，它依赖于 rime_ice，用九宫格别删 rime_ice.schema.yaml
        -  - schema: rime_ice               # 雾凇拼音（全拼）
        -  - schema: t9                     # 九宫格（仓输入法）
        +  # - schema: rime_ice               # 雾凇拼音（全拼）
        +  # - schema: t9                     # 九宫格（仓输入法）
           - schema: double_pinyin          # 自然码双拼
        -  - schema: double_pinyin_abc      # 智能 ABC 双拼
        -  - schema: double_pinyin_mspy     # 微软双拼
        -  - schema: double_pinyin_sogou    # 搜狗双拼
        -  - schema: double_pinyin_flypy    # 小鹤双拼
        -  - schema: double_pinyin_ziguang  # 紫光双拼
        +  # - schema: double_pinyin_abc      # 智能 ABC 双拼
        +  # - schema: double_pinyin_mspy     # 微软双拼
        +  # - schema: double_pinyin_sogou    # 搜狗双拼
        +  # - schema: double_pinyin_flypy    # 小鹤双拼
        +  # - schema: double_pinyin_ziguang  # 紫光双拼
         
         
         # 菜单
         menu:
        -  page_size: 5  # 候选词个数
        +  page_size: 9  # 候选词个数
           # alternative_select_labels: [ ①, ②, ③, ④, ⑤, ⑥, ⑦, ⑧, ⑨, ⑩ ]  # 修改候选项标签
           # alternative_select_keys: ASDFGHJKL  # 如编码字符占用数字键，则需另设选字键
         
        @@ -188,12 +188,12 @@ key_binder:
             - { when: has_menu, accept: equal, send: Page_Down }
         
             # 翻页 , .
        -    # - { when: paging, accept: comma, send: Page_Up }
        -    # - { when: has_menu, accept: period, send: Page_Down }
        +    - { when: paging, accept: comma, send: Page_Up }
        +    - { when: has_menu, accept: period, send: Page_Down }
         
             # 翻页 [ ]
        -    # - { when: paging, accept: bracketleft, send: Page_Up }
        -    # - { when: has_menu, accept: bracketright, send: Page_Down }
        +    - { when: paging, accept: bracketleft, send: Page_Up }
        +    - { when: has_menu, accept: bracketright, send: Page_Down }
         
             # 两种按键配置，鼠须管 Control+Shift+4 生效，小狼毫 Control+Shift+dollar 生效，都写上了。
             # numbered_mode_switch:
        diff --git a/double_pinyin.schema.yaml b/double_pinyin.schema.yaml
        index 154adbd..77e65e4 100644
        --- a/double_pinyin.schema.yaml
        +++ b/double_pinyin.schema.yaml
        @@ -35,6 +35,7 @@ schema:
         switches:
           - name: ascii_mode
             states: [ 中, Ａ ]
        +    reset: 1
           - name: ascii_punct  # 中英标点
             states: [ ¥, $ ]
           - name: traditionalization
        EOF
        ```
- references
  - [fcitx5-rime个人配置以及踩坑点解决简要](https://blacksand.top/2021/10/18/fcitx5-rime%E4%B8%AA%E4%BA%BA%E9%85%8D%E7%BD%AE%E4%BB%A5%E5%8F%8A%E8%B8%A9%E5%9D%91%E7%82%B9%E8%A7%A3%E5%86%B3%E7%AE%80%E8%A6%81/)

---

## Comments

### konosubakonoakua · 2024-09-21T19:27:44Z

## references
- [official CustomizationGuide](https://github.com/rime/home/wiki/CustomizationGuide)
- [iDvel/rime-ice](https://github.com/iDvel/rime-ice)

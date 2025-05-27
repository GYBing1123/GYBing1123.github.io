+++
date = '2025-05-27T20:50:47+08:00'
draft = false
title = 'Archlinux 的配置'
categories = ["linux"]
tags = ["archlinux"]
+++
# 添加archlinuxcn仓库
archlinuxcn 是为 Arch Linux 系统用户提供额外软件包的社区仓库。
## 1.添加源
```bash
通过vim /etc/pacman.conf命令，在pacman的配置文件中加入以下文字。
## 北京大学 (北京) (ipv4, ipv6, http, https)
## Added: 2023-02-26
[archlinuxcn]
Server = https://mirrors.pku.edu.cn/archlinuxcn/$arch 
```

## 2.安装 GPG 密钥
archlinuxcn 仓库的软件包使用 GPG 密钥进行签名验证，所以需要安装相应的 GPG 密钥

```bash
sudo pacman -Sy && sudo pacman -S archlinuxcn-keyring
```

---
# 软件安装
## 1. yay
Yay 是一个在 Arch Linux 及其衍生发行版（如 Manjaro）上广泛使用的 AUR（Arch User Repository）助手工具，它结合了 pacman 的功能并对其进行扩展，为用户提供了更加便捷的软件包管理体验。
```bash
pacman -S yay
```

## 2. Neofetch
```bash
yay -S neofetch
```


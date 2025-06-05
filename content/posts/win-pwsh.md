+++
date = '2025-05-26T16:27:22+08:00'
draft = false
title = 'Windows Powershell 的配置和优化'
categroies = ["windows"]
tags = ["windows", "powershell"]
+++

Windows 是我工作环境的操作系统，在接触了 Linux 的 shell 后，就觉得 Windows 的 cmd 功能很有限，而且许多操作的逻辑和 shell 是完全不同的。
Win 10 系统自带的 powershell 可以实现部分 shell 功能， 我之前一直使用的也是系统自带的 powershell，之后了解到 windows 的powershell 经过配置，也可以实现复杂的 shell 功能。所以才有了这篇博客，记录学习过程。

---
# 需要的软件
- Windows Terminal
- PowerShell
- oh-my-posh

---
# 安装
## 1. 下载 Windows Terminal 和 Powershell
在 Microsoft Store 中下载 `Windows Terminal` 和 `Powershell`
- `Windows Terminal` 一般来说，系统都会自带；
- `Powershell` 系统自带的叫做 `windows powershell` 版本较为老旧，有需要功能不支持；

## 2. 下载字体
需要下载一种 `Nerd Font`,这种字体支持图标文字，可以显示多种特殊图标。
推荐以下字体：
- JetBrainsMono Nerd Font
- Maole Mono
- 霞鹭文楷


## 3. 下载 Oh-My-Posh
`oh-my-posh` 是一种命令行提示符美化工具。它支持多种主题、图标和自定义配置，提升使用体验。

可以通过 `winget` 的方式安装。
（更多人推荐 scoop 安装方式，但我个人觉得 winget 更傻瓜一些。可能我使用了 admin 账户的问题 scoop 本身运行并不顺手）
```bash
# 搜寻软件
winget search oh-my-posh

# 根据搜索的结果安装需要的版本
winget install JanDeDobbeleer.OhMyPosh
```

---
# 初始化 Powershell
1. 以管理员的身份运行 Powershell；
2. 将 `属性-终端-默认终端应用程序` 设置为 `Windows 终端`，保存并关闭窗口；
3. 再次打开 Powershell ，在  `设置-启动-默认配置文件` 中选择 `Powershell` ；
4. 选择 `以管理员身份运行此配置文件` ;
5. 在 `外观-字体` 菜单中选择一种 `Nerd Font`;

---
# 配置 `oh-my-posh`
1. 确定使用的shell
```bash
oh-my-posh get shell
# 我的输出为pwsh，即powershell
```

---
2. 编辑配置文件
2.1 创建并打开 `powershell` 的配置文件
```bash
# 创建
New-Item -Path $PROFILE -Type File -Force

# 编辑
notepad $PROFILE
```

2.2 添加代码到配置文件
```bash
oh-my-posh init pwsh | Invoke-Expression
```

2.3 重新加载配置文件
```bash
. $PROFILE
```

---
3. 个性化配置
3.1 主题
编辑配置文件，添加主题
```bash
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH/jandedobbeleer.omp.json" | Invoke-Expression
```

- `jandedobbeleer.omp.json` 为主题名，
- 主题文件保存地址为 `C:\Users\Administrator\AppData\Local\Programs\oh-my-posh\themes` 
- 主题文件保存地址可以写为环境变量 `POSH_THEMES_PATH` 

---
# 配置 `powershell`
`Powershell` 配置的流程：
-  使用 `Install-Module` 命令，从 `PowerShell` 库安装模块。
-  在配置文件中通过 `Import-Module` 命令加载配置。

## 1. 配置图标库
1.1 在 `pwsh` 中执行以下命令
```bash
Install-Module -Name Terminal-Icons`
```

1.2 在配置文件  `$PROFILE` 中添加代码，加载配置
```bash
Import-Module -Name Terminal-Icons
```

---
# `powershell` 使用技巧
## 1 分屏

|按键|作用|
|:--:|:--|
|`SHIFT+ALT+'-'`|上下分屏|
|`SHIFT+ALT+'+'`|左右分屏|
|`CTRL+SHIFT+w`|关闭当前窗口|
|`ALT+方向键`|选择窗口|




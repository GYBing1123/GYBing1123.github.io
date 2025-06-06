+++
date = '2025-05-27T15:53:30+08:00'
draft = false
title = 'ArchLinux的安装'
categories = ["linux"]
tags = ["archlinux", "install"]
+++
# 正式安装之前
---
## 1. 准备工作
### 1.1 确认是 `UEFI` 还是 `BIOS`
- 输入以下命令，查看是否有输出
```bash
ll /sys/firmware/efi/efivars
# 1. 若有输出，说明是UEFI模式
# 2. 若没有输出，说明是BIOS模式
```
### 1.2 关闭reflector.service
2020 年，Archlinux 安装镜像中加入了 reflector 服务，它会自己更新 mirrorlist（软件包管理器 pacman 的软件源）。
- 查看服务状态
```bash
systemctl status reflector.service 
```

- 若服务开启中，则手动关闭它
```bash
systemctl stop reflector.service
systemctl disable reflector.service
```

---
## 2. 网络
### 2.1 连接网络
- 直连无需操作，只需要确认网卡开启即可

- 无线可通过iwctl 进行连接
```bash
iwctl # 进入交互式命令行
device list # 列出无线网卡设备名，比如无线网卡看到叫 wlan0
station wlan0 scan # 扫描网络
station wlan0 get-networks # 列出所有 wifi 网络
station wlan0 connect wifi-name # 进行连接，注意这里无法输入中文。回车后输入密码即可
exit # 连接成功后退出
```

### 2.2 测试网络
```bash
ping -c 5 www.baidu.com
```

---
## 3. 设置时区
- 查看当前的时间设置
```bash
timedatectl
```

- 设置时区
```bash
timedatectl set-timezone Asia/Shanghai
```

- 开启 NTP 同步
```bash
timedatectl set-ntp true
```

# 系统安装
---
## 1. 分区
### 1.1 查看磁盘情况
```bash
lsblk -p
```

### 1.2 使用cfdisk工具进行分区
-  UEFI模式下的分区情况（推荐）

|挂载点|分区|类型|建议大小|我的使用|
|:--:|:--:|:--:|:--:|:--:|
|`/mnt/boot`|`/dev/<boot>`|`EFI`|`>=300Mib`|`1G`|
|`[SWAP]`|`/dev/<swap>`| `Linux swap`|`=512Mib`|`4G`|
|`/mnt`|`/dev/<root>`|`Linux Filesystem`| |`2/3`|
|`/mnt/home`|`/dev/<home>`|`Linux Filesystem`| |`1/3`|


- BIOS模式下的分区情况（推荐），与UEFI相比没有/boot分区

---
## 2. 格式化
以UEFI模式为例
- 对boot分区进行格式化
```bash
# -n 设置标签
mkfs.fat -F 32 -n ARCHBOOT /dev/sda1
```

- 对swap分区进行格式化
```bash
mkswap /dev/sda2
```

- 对/分区进行格式化
```bash
# -L 设置标签
mkfs.xfs -L ARCHROOT /dev/sda3
```

- 对/home分区进行格式化
```bash
mkfs.xfs -L ARCHHOME /dev/sda4
```

---
## 3. 挂载
- 挂载根分区
```bash
`mount /dev/sda3 /mnt`
```

>安装阶段，程序运行与独立的临时环境(Initramfs)，此时系统尚未加载目标磁盘的文件系统。将跟目录挂载到/mnt是为了
>1. 为后续的系统文件提供写入目标
>2. 建立于目标磁盘的文件系统关联
>3. 避免与临时环境的/目录冲突
>4. /mnt仅在安装阶段作为临时过渡使用
>基础配置完成后，需要使用 `arch-chroot /mnt` 命令,切换到 `/mnt` 为根的系统，进行下一步配置。

- 建立其他目录
```bash
# 建立/boot目录
mkdir /mnt/boot

# 建立用户目录
mkdir /mnt/home
```

- 挂载其他分区
```bash
# 挂载BOOT分区
mount /dev/sda1 /mnt/boot

# 挂载用户分区
mount /dev/sda4 /mnt/home

挂载swap分区
swapon /dev/sda2
```

---
## 4. 安装软件
### 4.1 更新pacman源
- 刷新pacman包库
```bash
pacman -Syy
```

- 下载 pacman-mirrorlist 软件包
pacman-mirrorlist 包含了 Arch Linux 软件仓库镜像源的列表
```bash
pacman -S pacman-mirrorlist
```

- 使用grep语法将中国区的镜像源写入系统pacman源
1. 进入pacman源目录，pacman-mirrorlist软件包下载的mirrorlist.pacman文件也在该目录
```bash
cd /etc/pacman.d/
```

2. 使用grep语法
```bash
grep China -A 30 mirrorlist.pacman>mirrorlist
# -A 是 grep 的一个选项，代表 “after”，即输出匹配行之后的若干行。
```

3. 编辑mirrorlist文件，取消注释
```bash
vim mirrorlist
```

4. 再次刷新pacman包库
```bash
pacman -Syy
```

### 4.2 使用pacstrap安装基础包
pacstrap 用于将软件包安装到指定的系统分区中，这里是安装到临时根目录/mnt中
```bash
pacstrap /mnt base base-devel linux linux-firmware networkmanager vim \
man-db man-pages texinfo bash-completion vi sudo openssh \
dosfstools xfsprogs lvm2
```
- `base` `base-devel` `linux` `linux-firmware` linux系统的最小安装软件
- `networkmanager` 网络管理工具
- `vim` 文本编辑器
- `man-db` `man-pages` `texinfo` 帮助文档
- `bash-completion` 命令行补全工作
- `vi` 文本编辑器，配合sudo使用
- `sudo` 配合vi，可以使用visudo编辑超级用户
- `dosfstools` 用于管理 FAT 文件系统（包括 FAT12、FAT16 和 FAT32）的工具
- `xfsprogs` 用于管理 XFS 文件系统的工具集
- `lvm2`  Linux 下用于实现逻辑卷管理（Logical Volume Management，LVM）的工具集

---
## 5. 生成fstab文件
genfstab 是用于生成文件系统表(/etc/fstab)的工具。
这里将根据/mnt目录下的挂载情况，生成/etc/fstab文件内容，并将它追加到/mnt/etc/fstab中
```bash
genfstab -U /mnt>>/mnt/etc/fstab
# -U 使用UUID标识进行分区

# 查看配置
cat /mnt/etc/fstab
```

---
## 6. arch-chroot
使用 `arch-chroot /mnt` 改变根目录，进入安装好的系统。结束预安装

# 系统配置
---
## 1. 时区和语言
- 时区设置
```bash
# 1.通过软连接设置时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 2.把系统时间同步到硬件时钟
hwclock --systohc
```

- 语言设置
```bash
# 1.进入配置文件locale.gen设置哪些本地化环境会被生成。
vim /etc/locale.gen    # 去掉en_US.UTF-8 UTF-8的注释
# 2.执行文件，生成本地环境
locale-gen
# 3.写入配置到locale.conf，设置系统默认的本地化环境
echo "LANG=en_US.UTF-8">/etc/locale.conf
```

---
## 2. 设置hostname
```bash
echo "ArchLinux.GYBing" > /etc/hostname
```

---
## 3. 设置账户
- 通过 `passwd` 命令为root用户设置密码

- 添加管理员账户
ArchLinux默认root用户不能通过ssh登录。所以要提前设置好一个有sudo权限的普通用户，普通用户通过ssh登录，并结合sudo命令对服务器进行远程管理。
1. 添加账户
```bash
useradd -m -G wheel -s /bin/bash gybing
# -m：此选项的作用是在创建用户时，同时为该用户创建家目录。
# -G 选项用于指定用户所属的附加组,wheel组即为可以执行sudo命令的组。
# -s 选项用于设置用户的默认登录 shell。
```

2. 为用户设置密码
```bash
passwd gybing
```

3. 为用户添加sudo权限
```bash
visudo
# 进入sudo的配置文件后，取消 wheel 组的注释。
```

---
## 4. 设置网络和ssh自启
- 网络自启
```bash
systemctl enable NetworkManager.service
```

- ssh自启
```bash
 systemctl enable sshd.service
 ```

---
## 5. grub
### 5.1 安装CPU微码
- 查看 CPU 信息
```bash
lscpu
```

- 根据 CPU 信息选择对应的微码
```bash
# 1. Intel
pacman -S intel-ucode
# 2. AMD
pacman -s amd-ucode
```

### 5.2 安装grub
#### 5.2.1 单系统
- 软件安装
```bash
pacman -S grub efibootmgr
# grub grub工具
# efibootmgr UEFI启动管理器的命令行工具，BIOS引导不需要。
```

- 配置grub
安装引导
```bash
# 1.安装GRUB引导程序
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# --target 用于指定目标平台和环境，这里是x86_64架构，并且采用了UEFI启动模式
# --efi-directory 用于指定EFI系统分区的挂载点
# --bootloader-id 用于指定UEFI固件中显示的标识符

# 2.安装BIOS引导
grub-install --target=i386-pc /dev/sda
#注意这里的/dev/sda是要安装的磁盘，不是分区
```

生成GRUB配置文件
```bash
grub-mkconfig -o /boot/grub/grub.cfg
# -o 即--output，指定输出目录
```

#### 5.2.2 多系统
- 软件安装
```bash
pacman -S grub efibootmgr

pacman -S os-prober ntfs-3g
# os-prober 用于检测系统中的其他操作系统
# ntfs-3g 在Linux系统中读取NTFS分区的驱动程序
```

- 配置grub
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

# 使用vim /etc/default/grub命令，对/etc/default/grub文件进行配置
# 去掉GRUB_DISABLE_OS_PROBER=false行的注释

grub-mkconfig -o /boot/grub/grub.cfg
```

---
## 6. 取消临时挂载，并完成安装
- 离开chroot
```bash
exit
```

- 卸载硬盘
```bash
umount -R /mnt
swapoff /dev/sda2
```

- 重启,进入系统
```bash
reboot
```

title: Mac T2 芯片机型安装 Arch Linux 教程
date: '2026-10-08 22:43:47'
updated: '2026-10-08 22:43:51'
tags:
  - Blog
  - linux
categories:
  - 每日一记
---
> 基于 [t2linux wiki 官方文档](https://wiki.t2linux.org/distributions/arch/installation/) 整理，
> 适用于搭载 T2 安全芯片的 MacBook Pro / MacBook Air / iMac 等机型，双系统（macOS + Arch Linux）场景。

## 1. 准备工作

- **备份数据**：用 Time Machine 或其他方式备份 macOS 上的重要数据，整个过程涉及分区操作，有风险。
- **U 盘**：至少 1GB 容量的 USB 驱动器（USB-A 或 USB-C，视你 Mac 接口而定）。
- **确认机型支持情况**：访问 [设备支持状态页](https://wiki.t2linux.org/state/) 确认你的机型/功能支持程度。

---

## 2. 分区（macOS 端）

1. 打开 **磁盘工具**（Disk Utility）。
2. 选中主硬盘 → 点击 **"分区"**（Partition）。
3. 点击饼图下方的 **"+"** → 弹窗中务必选择 **"Add Partition"（添加分区）**，**不要选 "Add Volume"（添加卷）**，否则会建成 APFS 容器内的卷而不是独立分区。
4. 命名分区（如 `Linux`），格式随意（exFAT 或 Mac OS 扩展均可，后续会被 Linux 重新格式化）。
5. 分配好大小后确认。

> ⚠️ **关于现有 EFI 分区**：Mac 磁盘上已经存在一个约 300MB 的 EFI 系统分区（通常是 `/dev/nvme0n1p1`），**全程不需要新建或格式化它**，Arch 安装时直接复用即可。只有计划三系统（还要装 Windows）时才需要额外规划 EFI 分区，参考 [Windows 三系统指南](https://wiki.t2linux.org/guides/windows/)。

### 关闭 Secure Boot，允许外部启动

1. 重启进入恢复模式：开机时按住 **Cmd + R**。
2. 进入 **工具程序 → 启动安全性实用工具**。
3. 选择 **"不安全"/"降低安全性"**，并勾选允许从外部介质启动。

### （可选）提前复制 WiFi/蓝牙固件到 EFI 分区

T2 Mac 的 WiFi 是博通芯片，需要闭源固件，live 环境默认没有，提前复制可以避免安装阶段没有网络：

```bash
curl -sL https://wiki.t2linux.org/tools/firmware.sh | bash -s copy_to_efi
```

---

## 3. 制作安装 U 盘

下载 T2 专用的 Arch Linux ISO（**不能用官方原版 Arch ISO，没有 T2 内核补丁装不了**）：

```
https://github.com/t2linux/archiso-t2/releases/latest
```

用 `dd` 或 balenaEtcher 写入 U 盘。

---

## 4. 进入 Live 环境

插入 U 盘，重启，按住 **Option (⌥)** 键，选择 U 盘对应的 **"EFI Boot"** 启动项进入 live 环境。

### 初始化密钥环（强烈建议第一步就做，避免后续 pacstrap 报错）

live 环境启动后，`pacman-init.service` 会在后台自动初始化密钥环，但偶尔会有竞态问题导致后续安装报 `keyring is not writable` 或签名 "unknown trust" 错误。建议先手动确认一遍：

```bash
systemctl status pacman-init.service   # 确认是 inactive (dead)，不是 activating
pacman-key --init
pacman-key --populate archlinux
```

### 确认系统时间正确

GPG 签名校验对系统时间敏感，时间错乱会导致各种"signature unknown trust"/"corrupted package"报错：

```bash
timedatectl set-ntp true
timedatectl status    # 确认 System clock synchronized: yes
```

---

## 5. 联网（重要，常见卡点）

Live 环境默认**没有 WiFi**（博通芯片需要闭源固件）。按方便程度选择：

### 方法一：有线网络（最简单）
USB 转以太网适配器，插上基本免驱动自动获取 IP。

### 方法二：手机 USB 共享网络
数据线连接手机，开启 USB 网络共享/个人热点。

### 方法三：使用提前复制好的固件（如果第 2 步做过）

```bash
mount /dev/nvme0n1p1 /mnt
bash /mnt/firmware.sh get_from_efi
umount /mnt

# 重启iwd服务
systemctl restart iwd.service

iwctl
# 在 iwctl 交互界面里：
station wlan0 connect 你的WiFi名
```

---

## 6. 手动分区确认与格式化

### 查看分区情况

```bash
lsblk
# 或者针对整个磁盘（不要带分区号）
gdisk -l /dev/nvme0n1
```

> ⚠️ **易错点**：一定要对整个磁盘 `/dev/nvme0n1` 执行 `gdisk -l`，而不是对某个分区（如 `/dev/nvme0n1p1`）执行。对分区本身执行会被工具误读成 "Disklabel type: dos" 这种错误提示，这只是文件系统引导扇区的误判，不代表磁盘真的损坏。

正常情况下磁盘结构类似：

| 分区 | 内容 | 类型码 | 操作 |
|---|---|---|---|
| `nvme0n1p1` | EFI 系统分区（约 300MB） | `EF00` | **不格式化**，直接复用 |
| `nvme0n1p2` | macOS（APFS） | `AF00` | 不要动 |
| `nvme0n1p3` | 之前划出的 Linux 分区 | `8300`（装完后会是）| 格式化 |

### 格式化 Linux 分区（跳过 EFI 分区）

**ext4 方案：**
```bash
mkfs.ext4 /dev/nvme0n1p3
```

**btrfs 方案（如果偏好 btrfs 的快照/压缩特性）：**
```bash
mkfs.btrfs -L archlinux /dev/nvme0n1p3

# 创建子卷（可选但推荐）
mount /dev/nvme0n1p3 /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@var_log
umount /mnt
```

⚠️ **全程不要对 `nvme0n1p1`（EFI 分区）执行任何 `mkfs` 命令。**

---

## 7. 挂载分区

**ext4 方案：**
```bash
mount /dev/nvme0n1p3 /mnt
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

**btrfs 方案（按子卷挂载）：**
```bash
mount -o noatime,compress=zstd,subvol=@ /dev/nvme0n1p3 /mnt
mkdir -p /mnt/{home,.snapshots,var/log,boot}
mount -o noatime,compress=zstd,subvol=@home /dev/nvme0n1p3 /mnt/home
mount -o noatime,compress=zstd,subvol=@snapshots /dev/nvme0n1p3 /mnt/.snapshots
mount -o noatime,compress=zstd,subvol=@var_log /dev/nvme0n1p3 /mnt/var/log
mount /dev/nvme0n1p1 /mnt/boot
```

> 这里的关键是把**现有 EFI 分区挂载到 `/mnt/boot`**，而不是重新创建它——引导程序只是往里面新增一个 Linux 的引导项，macOS 原有的引导文件不会被删除或覆盖。

---

## 8. 安装基础系统

### 方式 A：pacstrap（更接近原生 Arch 体验，推荐）

```bash
pacstrap /mnt base linux-t2 linux-t2-headers arch-mact2-mirrorlist \
  arch-mact2-rankmirrors apple-t2-audio-config apple-bcm-firmware-fetcher \
  linux-firmware iwd grub efibootmgr t2fanrd btrfs-progs
```
（如果用 systemd-boot 而非 GRUB，去掉 `grub efibootmgr`；如果用 ext4 而非 btrfs，去掉 `btrfs-progs`）

安装前需要把 T2 的仓库加进 `/mnt/etc/pacman.conf`：

```bash
vi /mnt/etc/pacman.conf
```
加入：
```ini
[arch-mact2]
Include = /etc/pacman.d/arch-mact2-mirrorlist
SigLevel = Never
```

### 方式 B：t2strap（更省事，自动配置好仓库）

```bash
t2strap /mnt base linux-firmware iwd grub efibootmgr
```

---

## 9. 系统基础配置（时区、语言、主机名）

### 生成 fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab   # 检查无误，确认各分区 UUID 正确对应
```

### 进入 chroot

```bash
arch-chroot /mnt
```

### 设置时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

### 设置语言 locale

```bash
vi /etc/locale.gen
```
取消注释（去掉行首 `#`）：
```
zh_CN.UTF-8 UTF-8
en_US.UTF-8 UTF-8
```
生成：
```bash
locale-gen
```
设置系统默认语言：
```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

### 设置主机名

```bash
echo "archlinux-t2" > /etc/hostname
vi /etc/hosts
```
写入：
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux-t2.localdomain archlinux-t2
```

### 设置 root 密码

```bash
passwd
```

---

## 10. 新增用户与权限

```bash
useradd -m -G wheel -s /bin/bash user1
passwd user1
```
- `-m`：自动创建家目录
- `-G wheel`：加入 wheel 组（配合 sudo 权限）
- `-s /bin/bash`：默认 shell

### 配置 sudo 权限

```bash
pacman -S sudo
EDITOR=vim visudo
```

找到并取消注释：
```
%wheel ALL=(ALL:ALL) ALL
```

---

## 11. 安装引导程序（GRUB）

### 编辑 GRUB 配置文件

```bash
nano /etc/default/grub
```
找到 `GRUB_CMDLINE_LINUX="quiet splash"` 这一行，加入以下内核参数（T2 芯片必需）：
```
GRUB_CMDLINE_LINUX="quiet splash intel_iommu=on iommu=pt pm_async=off"
```

### 安装 GRUB（⚠️ Mac 上必须加 `--removable`）

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --removable
```

> **为什么必须加 `--removable`**：Mac 的 Startup Manager（按住 Option 的启动选择界面）不读取 NVRAM 里 `efibootmgr` 写入的条目，只扫描每个分区固定的 `/EFI/BOOT/BOOTX64.EFI` 路径。不加这个参数，装完按 Option 只会看到 macOS，看不到 Linux 引导项。
>
> 如果已经装过忘记加这个参数，可以不重装，直接补一份文件：
> ```bash
> mkdir -p /boot/EFI/BOOT
> cp /boot/EFI/GRUB/grubx64.efi /boot/EFI/BOOT/BOOTX64.EFI
> ```

### 生成 GRUB 配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
确认输出中包含：
```
Found linux image: /boot/vmlinuz-linux-t2
Found initrd image: /boot/initramfs-linux-t2.img
```
如果没有这两行，说明 `linux-t2` 内核包没装成功（常见于之前密钥环报错导致包被跳过），需要补装：
```bash
pacman -S linux-t2 linux-t2-headers
```

### 启用风扇控制服务

```bash
systemctl enable t2fanrd
```

---

## 12. 完成安装并重启

```bash
exit
umount -R /mnt
reboot
```
拔掉 U 盘，开机按住 **Option (⌥)**，应该能同时看到 macOS 和 Arch Linux 的引导选项。

---

## 13. 安装 Deepin 桌面环境

```bash
sudo pacman -S deepin deepin-extra
```
- `deepin`：核心桌面组件
- `deepin-extra`：原生应用（截图、相册、音乐播放器等）

### 安装登录管理器

```bash
sudo pacman -S lightdm lightdm-deepin-greeter
sudo systemctl enable lightdm
```

---

## 14. 中文环境与输入法

### 安装中文字体

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji
fc-cache -fv
```

### 安装中文输入法（fcitx5）

```bash
sudo pacman -S fcitx5-im fcitx5-chinese-addons
```
编辑 `~/.pam_environment` 或 `/etc/environment`，加入：
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```
注销重新登录生效，系统托盘应出现 fcitx5 图标，右键配置添加拼音/五笔等输入法。

---

## 参考链接

- [t2linux wiki 安装指南原文](https://wiki.t2linux.org/distributions/arch/installation/)
- [Pre-Install 准备指南](https://wiki.t2linux.org/guides/preinstall/)
- [Basic Setup 后续配置](https://wiki.t2linux.org/guides/postinstall/)
- [WiFi 与蓝牙指南](https://wiki.t2linux.org/guides/wifi-bluetooth/)
- [风扇控制指南](https://wiki.t2linux.org/guides/fan/)
- [卸载指南](https://wiki.t2linux.org/guides/uninstall/)
- [设备支持状态](https://wiki.t2linux.org/state/)

# 第12章：Linux驱动管理

## 12.1 驱动程序概述

### 什么是驱动程序？

驱动程序是操作系统与硬件设备之间的桥梁，负责：
- 硬件初始化
- 数据传输
- 中断处理
- 电源管理

### Linux驱动架构

```
用户空间应用
    ↓
系统调用接口
    ↓
虚拟文件系统(VFS)
    ↓
设备驱动程序
    ↓
硬件设备
```

### 驱动类型

**字符设备驱动**：
- 顺序访问数据
- 示例：串口、键盘、鼠标

**块设备驱动**：
- 随机访问数据
- 示例：硬盘、U盘、SD卡

**网络设备驱动**：
- 网络数据传输
- 示例：网卡、WiFi

## 12.2 内核模块

### 什么是内核模块？

内核模块是可以动态加载和卸载的内核代码，无需重启系统。

### 查看已加载模块

```bash
# 列出所有已加载模块
lsmod

# 查看特定模块
lsmod | grep nvidia

# 模块详细信息
modinfo nvidia
modinfo e1000e  # Intel网卡驱动

# 模块依赖关系
modprobe --show-depends nvidia
```

### 加载和卸载模块

```bash
# 手动加载模块
sudo modprobe module_name

# 加载时传递参数
sudo modprobe module_name param1=value1

# 卸载模块
sudo modprobe -r module_name
sudo rmmod module_name  # 强制卸载（不推荐）

# 示例：加载USB存储模块
sudo modprobe usb-storage

# 卸载模块
sudo modprobe -r usb-storage
```

### 自动加载模块

```bash
# 开机自动加载（/etc/modules）
sudo nano /etc/modules
# 添加一行：module_name

# 模块参数配置（/etc/modprobe.d/）
sudo nano /etc/modprobe.d/module_name.conf
# 内容：options module_name param=value

# 示例：配置声卡模块
echo "options snd-hda-intel model=auto" | sudo tee /etc/modprobe.d/alsa.conf
```

### 黑名单模块

```bash
# 禁止加载某个模块
sudo nano /etc/modprobe.d/blacklist.conf

# 添加：
blacklist module_name
blacklist nouveau  # 禁用开源NVIDIA驱动

# 更新initramfs
sudo update-initramfs -u
```

### 模块信息

```bash
# 查看模块详细信息
modinfo module_name

# 输出示例：
# filename:       /lib/modules/.../module.ko
# version:        1.0
# license:        GPL
# description:    ...
# author:         ...
# depends:        other_modules
# parm:           parameter_name:description

# 查看模块使用情况
lsmod | grep module_name

# 查看模块参数
cat /sys/module/module_name/parameters/*
```

## 12.3 设备文件

### /dev目录

Linux中一切皆文件，硬件设备也表示为文件。

```bash
# 查看设备文件
ls -l /dev

# 常见设备文件：
/dev/sda        # 第一块SATA硬盘
/dev/sdb1       # 第二块硬盘的第一个分区
/dev/nvme0n1    # NVMe SSD
/dev/tty        # 终端设备
/dev/null       # 空设备
/dev/zero       # 零设备
/dev/random     # 随机数生成器
/dev/urandom    # 伪随机数生成器
/dev/input/*    # 输入设备
/dev/video0     # 摄像头
```

### 设备类型

```bash
# 字符设备（c）
crw-rw-rw- 1 root root 1, 3 /dev/null

# 块设备（b）
brw-rw---- 1 root disk 8, 0 /dev/sda

# 主设备号和次设备号
# 8, 0 表示：主设备号=8，次设备号=0
```

### udev - 设备管理器

udev负责动态创建和管理设备文件。

```bash
# udev规则目录
ls /etc/udev/rules.d/
ls /lib/udev/rules.d/

# 查看udev数据库
udevadm info /dev/sda

# 监控udev事件
udevadm monitor

# 触发udev重新扫描
sudo udevadm trigger

# 重新加载udev规则
sudo udevadm control --reload-rules
```

### 自定义udev规则

```bash
# 创建udev规则
sudo nano /etc/udev/rules.d/99-custom.conf

# 示例1：USB设备固定设备名
# 查看设备信息
udevadm info -a -p $(udevadm info -q path -n /dev/sdb)

# 规则示例：
SUBSYSTEM=="block", ATTRS{idVendor}=="0930", ATTRS{idProduct}=="6545", SYMLINK+="my_usb"

# 示例2：网卡重命名
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="00:11:22:33:44:55", NAME="lan0"

# 重新加载规则
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## 12.4 显卡驱动

### NVIDIA驱动

```bash
# 查看NVIDIA显卡
lspci | grep -i nvidia

# 检查推荐驱动
ubuntu-drivers devices

# 自动安装推荐驱动
sudo ubuntu-drivers autoinstall

# 或手动安装特定版本
sudo apt install nvidia-driver-535

# 查看NVIDIA驱动版本
nvidia-smi

# 配置NVIDIA设置
nvidia-settings

# 切换到集成显卡（节能）
sudo prime-select intel

# 切换到独立显卡（性能）
sudo prime-select nvidia

# 查看当前使用的GPU
prime-select query
```

**从官网安装NVIDIA驱动**：

```bash
# 1. 禁用nouveau驱动
sudo nano /etc/modprobe.d/blacklist-nouveau.conf
# 添加：
blacklist nouveau
options nouveau modeset=0

sudo update-initramfs -u
# 重启

# 2. 安装依赖
sudo apt install build-essential

# 3. 下载驱动
wget https://.../.../NVIDIA-Linux-x86_64-xxx.xx.run

# 4. 停止图形界面
sudo systemctl stop gdm  # 或lightdm/sddm

# 5. 安装驱动
sudo bash NVIDIA-Linux-x86_64-xxx.xx.run

# 6. 重启
sudo reboot
```

### AMD驱动

```bash
# 查看AMD显卡
lspci | grep -i amd

# 开源驱动（默认已安装）
# AMDGPU和Radeon驱动已包含在内核中

# 安装AMD官方驱动（AMDGPU-PRO）
# 从AMD官网下载驱动包
tar -xf amdgpu-pro-xxx.tar.xz
cd amdgpu-pro-xxx
./amdgpu-install

# 查看AMD GPU状态
radeontop  # 需安装
```

### Intel集成显卡

```bash
# Intel驱动已包含在内核中（i915模块）
lsmod | grep i915

# 安装Intel GPU工具
sudo apt install intel-gpu-tools

# 监控GPU
sudo intel_gpu_top
```

## 12.5 网卡驱动

### 有线网卡

```bash
# 查看网卡硬件
lspci | grep -i ethernet

# 查看网卡驱动
ethtool -i eth0

# 查看驱动模块
lspci -k | grep -A 3 -i ethernet

# 常见网卡驱动：
# - e1000e: Intel千兆网卡
# - r8169: Realtek网卡
# - igb: Intel多端口网卡
# - tg3: Broadcom网卡

# 加载网卡驱动
sudo modprobe e1000e

# 查看网卡状态
ip link show
ethtool eth0
```

### 无线网卡

```bash
# 查看无线网卡
lspci | grep -i wireless
lsusb | grep -i wireless

# 查看无线驱动
lspci -k | grep -A 3 -i wireless

# 常见无线驱动：
# - iwlwifi: Intel无线网卡
# - ath9k: Atheros网卡
# - rtl8xxxu: Realtek USB网卡
# - mt76: MediaTek网卡

# 安装Intel无线网卡固件
sudo apt install firmware-iwlwifi

# 安装Realtek固件
sudo apt install firmware-realtek

# 查看无线状态
iwconfig
nmcli device wifi list
```

### 网卡驱动问题排查

```bash
# 查看网卡是否识别
lspci | grep -i network
ip link show

# 查看dmesg日志
dmesg | grep -i network
dmesg | grep -i eth
dmesg | grep -i wlan

# 重新加载驱动
sudo modprobe -r driver_name
sudo modprobe driver_name

# 查看固件是否缺失
dmesg | grep -i firmware
```

## 12.6 声卡驱动

### ALSA - 高级Linux声音架构

```bash
# 查看声卡
lspci | grep -i audio
aplay -l  # 列出播放设备
arecord -l  # 列出录音设备

# 查看声卡驱动
lspci -k | grep -A 3 -i audio

# 常见声卡驱动：
# - snd-hda-intel: Intel HD Audio
# - snd-hda-codec-realtek: Realtek编解码器
# - snd-usb-audio: USB声卡

# 音量控制
alsamixer

# 测试声卡
speaker-test -c 2  # 双声道测试
```

### PulseAudio

```bash
# 查看PulseAudio状态
pulseaudio --check
systemctl --user status pulseaudio

# 重启PulseAudio
pulseaudio -k
pulseaudio --start

# 音量控制
pactl list sinks  # 列出输出设备
pactl set-sink-volume 0 50%  # 设置音量
```

### 声卡问题排查

```bash
# 检查声卡是否静音
alsamixer  # 按M取消静音

# 查看音频组
groups $USER
# 确保用户在audio组
sudo usermod -aG audio $USER

# 重新加载声卡驱动
sudo alsa force-reload

# 查看日志
dmesg | grep -i audio
journalctl -b | grep -i audio
```

## 12.7 打印机驱动

### CUPS - 通用Unix打印系统

```bash
# 安装CUPS
sudo apt install cups

# 启动CUPS服务
sudo systemctl start cups
sudo systemctl enable cups

# CUPS Web界面
# 浏览器访问: http://localhost:631

# 命令行管理打印机
lpstat -p -d  # 列出打印机
lpadmin  # 管理打印机

# 添加打印机
sudo lpadmin -p printer_name -E -v device_uri -P ppd_file

# 删除打印机
sudo lpadmin -x printer_name

# 打印测试页
echo "Test" | lpr
```

### 打印机驱动安装

```bash
# HP打印机（HPLIP）
sudo apt install hplip hplip-gui
hp-setup  # 图形化配置

# 其他品牌驱动
sudo apt install printer-driver-all

# 查看支持的打印机
lpinfo -m | grep "打印机型号"
```

## 12.8 其他常见驱动

### 蓝牙

```bash
# 安装蓝牙工具
sudo apt install bluez

# 启动蓝牙服务
sudo systemctl start bluetooth
sudo systemctl enable bluetooth

# 蓝牙管理
bluetoothctl
# 命令：
# power on
# scan on
# pair MAC_ADDRESS
# connect MAC_ADDRESS

# 查看蓝牙适配器
hciconfig
hcitool dev
```

### 触摸板

```bash
# 查看触摸板设备
xinput list | grep -i touchpad

# 触摸板设置
xinput list-props device_id

# 禁用触摸板
xinput disable device_id

# 启用触摸板
xinput enable device_id

# Synaptics触摸板配置
synclient -l  # 列出参数
synclient TapButton1=1  # 启用轻触点击
```

### 摄像头

```bash
# 查看摄像头设备
ls /dev/video*
v4l2-ctl --list-devices

# 测试摄像头
# 安装cheese
sudo apt install cheese
cheese

# 或使用ffplay
ffplay /dev/video0
```

## 12.9 驱动编译

### 从源码编译驱动

```bash
# 1. 安装编译工具
sudo apt install build-essential linux-headers-$(uname -r)

# 2. 下载驱动源码
wget https://.../driver.tar.gz
tar -xzf driver.tar.gz
cd driver

# 3. 编译
make

# 4. 安装
sudo make install

# 5. 加载模块
sudo modprobe module_name

# 6. 设置开机加载
echo "module_name" | sudo tee -a /etc/modules
```

### DKMS - 动态内核模块支持

```bash
# 安装DKMS
sudo apt install dkms

# 使用DKMS安装驱动
sudo dkms add -m module_name -v version
sudo dkms build -m module_name -v version
sudo dkms install -m module_name -v version

# 查看DKMS模块
dkms status

# 删除DKMS模块
sudo dkms remove -m module_name -v version --all
```

## 12.10 驱动问题排查

### 常用排查命令

```bash
# 查看内核日志
dmesg | tail -50
dmesg | grep -i error

# 查看系统日志
journalctl -xe
journalctl -b | grep -i driver

# 查看硬件识别
lspci -v
lsusb -v

# 查看已加载模块
lsmod

# 查看模块信息
modinfo module_name
```

### 常见驱动问题

**问题1：模块未加载**

```bash
# 查看是否被黑名单
cat /etc/modprobe.d/blacklist.conf

# 手动加载
sudo modprobe module_name

# 查看加载错误
dmesg | grep module_name
```

**问题2：固件缺失**

```bash
# 查看固件错误
dmesg | grep -i firmware

# 安装固件包
sudo apt install linux-firmware
sudo apt install firmware-linux-nonfree  # Debian
```

**问题3：驱动冲突**

```bash
# 查看使用驱动的进程
lsof /dev/device

# 卸载冲突模块
sudo modprobe -r conflicting_module

# 设置模块优先级
sudo nano /etc/modprobe.d/priority.conf
# 添加：
install module_name /sbin/modprobe --ignore-install module_name && /sbin/modprobe other_module
```

## 12.11 实用脚本

### 脚本1：驱动信息收集

```bash
#!/bin/bash
# driver_info.sh - 收集驱动信息

OUTPUT="driver_report.txt"

{
    echo "========== 已加载模块 =========="
    lsmod | head -20
    echo ""

    echo "========== 网卡驱动 =========="
    lspci -k | grep -A 3 -i network
    echo ""

    echo "========== 显卡驱动 =========="
    lspci -k | grep -A 3 -i vga
    echo ""

    echo "========== 声卡驱动 =========="
    lspci -k | grep -A 3 -i audio
    echo ""

    echo "========== USB设备 =========="
    lsusb
    echo ""

    echo "========== 内核日志（最近50行）=========="
    dmesg | tail -50

} > $OUTPUT

echo "驱动信息已保存到: $OUTPUT"
```

### 脚本2：驱动重载工具

```bash
#!/bin/bash
# reload_driver.sh - 重新加载驱动

if [ $# -ne 1 ]; then
    echo "用法: $0 <模块名>"
    exit 1
fi

MODULE=$1

echo "卸载模块: $MODULE"
sudo modprobe -r $MODULE

sleep 2

echo "重新加载模块: $MODULE"
sudo modprobe $MODULE

echo "模块状态:"
lsmod | grep $MODULE

echo "内核日志:"
dmesg | tail -10
```

### 脚本3：驱动问题诊断

```bash
#!/bin/bash
# diagnose_driver.sh - 驱动问题诊断

echo "=== 驱动问题诊断 ==="
echo ""

# 检查硬件识别
echo "检查硬件识别..."
if ! lspci > /dev/null 2>&1; then
    echo "警告: 无法读取PCI设备"
fi

# 检查固件
echo "检查固件错误..."
FIRMWARE_ERRORS=$(dmesg | grep -i "firmware" | grep -i "failed\|error")
if [ ! -z "$FIRMWARE_ERRORS" ]; then
    echo "发现固件错误:"
    echo "$FIRMWARE_ERRORS"
fi

# 检查驱动加载失败
echo "检查驱动加载失败..."
MODULE_ERRORS=$(dmesg | grep -i "module" | grep -i "failed\|error")
if [ ! -z "$MODULE_ERRORS" ]; then
    echo "发现模块加载错误:"
    echo "$MODULE_ERRORS"
fi

# 检查网卡
echo "检查网络驱动..."
if ! ip link show > /dev/null 2>&1; then
    echo "警告: 网络接口异常"
fi

echo ""
echo "诊断完成！"
```

## 12.12 实践练习

### 练习1：查看和管理模块

```bash
# 1. 列出所有模块
lsmod

# 2. 查看特定模块信息
modinfo e1000e

# 3. 卸载并重新加载模块（以USB存储为例）
sudo modprobe -r usb_storage
lsmod | grep usb_storage
sudo modprobe usb_storage
lsmod | grep usb_storage
```

### 练习2：创建udev规则

```bash
# 1. 插入U盘并查看设备信息
lsblk
udevadm info -a -p $(udevadm info -q path -n /dev/sdb)

# 2. 创建自定义规则
sudo nano /etc/udev/rules.d/99-usb.conf
# 添加规则（根据你的设备调整）

# 3. 重新加载规则
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### 练习3：查看驱动状态

```bash
# 查看网卡驱动
lspci -k | grep -A 3 -i network
ethtool -i eth0

# 查看显卡驱动
lspci -k | grep -A 3 -i vga

# 查看声卡驱动
lspci -k | grep -A 3 -i audio
```

## 下一步

完成本章后，你应该：
- ✅ 理解Linux驱动架构
- ✅ 掌握内核模块管理
- ✅ 了解udev设备管理
- ✅ 会安装和配置常见驱动
- ✅ 能够排查驱动问题

**准备好了吗？让我们进入[第13章：内核原理](../13-内核原理/README.md)！**

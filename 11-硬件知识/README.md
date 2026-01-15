# 第11章：Linux硬件知识

## 11.1 硬件架构概述

### 计算机硬件组成

```
┌─────────────────────────────────────┐
│            计算机系统                │
├─────────────────────────────────────┤
│  CPU  │  内存  │  主板  │  存储设备 │
├─────────────────────────────────────┤
│  显卡  │  网卡  │  声卡  │  外设    │
└─────────────────────────────────────┘
```

### Linux与硬件交互

```
应用程序
    ↓
系统调用
    ↓
Linux内核
    ↓
设备驱动
    ↓
硬件设备
```

## 11.2 查看硬件信息

### lshw - 详细硬件信息

```bash
# 查看所有硬件（需root）
sudo lshw

# 简洁格式
sudo lshw -short

# HTML格式输出
sudo lshw -html > hardware.html

# 查看特定类型
sudo lshw -class processor  # CPU
sudo lshw -class memory     # 内存
sudo lshw -class disk       # 磁盘
sudo lshw -class network    # 网卡
sudo lshw -class display    # 显卡

# 安装lshw
sudo apt install lshw       # Ubuntu/Debian
sudo dnf install lshw       # Fedora
```

### lspci - PCI设备

```bash
# 列出所有PCI设备
lspci

# 详细信息
lspci -v
lspci -vv  # 更详细

# 显示树形结构
lspci -t

# 显示数字ID
lspci -nn

# 查看特定设备
lspci | grep -i vga       # 显卡
lspci | grep -i ethernet  # 网卡
lspci | grep -i audio     # 声卡

# 查看设备详细信息
lspci -s 00:02.0 -v  # 00:02.0 是设备ID
```

### lsusb - USB设备

```bash
# 列出USB设备
lsusb

# 详细信息
lsusb -v

# 树形结构
lsusb -t

# 查看特定设备
lsusb -d 046d:c52b  # 使用vendorID:productID
```

### dmidecode - BIOS/硬件信息

```bash
# 查看所有DMI信息
sudo dmidecode

# 查看特定类型
sudo dmidecode -t processor  # CPU
sudo dmidecode -t memory     # 内存
sudo dmidecode -t bios       # BIOS
sudo dmidecode -t system     # 系统信息
sudo dmidecode -t baseboard  # 主板

# 简洁输出
sudo dmidecode -q
```

### hwinfo - 综合硬件信息

```bash
# 安装hwinfo
sudo apt install hwinfo

# 查看所有硬件
sudo hwinfo

# 简洁模式
sudo hwinfo --short

# 查看特定硬件
sudo hwinfo --cpu
sudo hwinfo --memory
sudo hwinfo --disk
sudo hwinfo --network
```

## 11.3 CPU信息

### 查看CPU信息

```bash
# /proc/cpuinfo
cat /proc/cpuinfo

# 查看CPU型号
cat /proc/cpuinfo | grep "model name" | head -1

# 查看CPU核心数
nproc
lscpu | grep "^CPU(s):"

# 详细CPU信息
lscpu

# CPU架构
uname -m
arch
```

### lscpu - CPU详细信息

```bash
lscpu

# 主要输出：
# Architecture:        x86_64              # 架构
# CPU op-mode(s):      32-bit, 64-bit     # 操作模式
# CPU(s):              8                   # 总核心数
# Thread(s) per core:  2                   # 每核心线程数
# Core(s) per socket:  4                   # 每插槽核心数
# Socket(s):           1                   # 插槽数
# Model name:          Intel Core i7-8xxx # CPU型号
# CPU MHz:             2400.000            # 频率
# Cache:               8192 KB             # 缓存
```

### CPU频率管理

```bash
# 查看当前频率
cat /proc/cpuinfo | grep MHz

# 或使用cpufreq（需安装）
sudo apt install cpufrequtils

# 查看频率信息
cpufreq-info

# 查看可用的调速器
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors

# 设置性能模式
sudo cpufreq-set -g performance

# 设置节能模式
sudo cpufreq-set -g powersave

# 查看CPU温度（需lm-sensors）
sudo apt install lm-sensors
sudo sensors-detect  # 检测传感器
sensors              # 显示温度
```

### CPU压力测试

```bash
# 使用stress工具
sudo apt install stress

# CPU满载测试（4个worker）
stress --cpu 4 --timeout 60s

# 综合测试
stress --cpu 4 --io 2 --vm 2 --vm-bytes 128M --timeout 60s

# 使用stress-ng（更强大）
sudo apt install stress-ng
stress-ng --cpu 4 --timeout 60s --metrics-brief
```

## 11.4 内存信息

### 查看内存使用

```bash
# free命令
free -h              # 人类可读
free -m              # MB单位
free -g              # GB单位
free -s 1            # 每秒刷新

# /proc/meminfo
cat /proc/meminfo

# 查看总内存
grep MemTotal /proc/meminfo

# 查看可用内存
grep MemAvailable /proc/meminfo
```

### 内存详细信息

```bash
# dmidecode查看内存条信息
sudo dmidecode -t memory

# 主要信息：
# - Size: 8192 MB        # 容量
# - Type: DDR4           # 类型
# - Speed: 2400 MT/s     # 频率
# - Manufacturer: ...    # 厂商

# 查看内存插槽
sudo dmidecode -t memory | grep -i "size"

# 内存错误检测
sudo dmidecode -t memory | grep -i "error"
```

### 内存测试

```bash
# 安装memtester
sudo apt install memtester

# 测试1GB内存5次
sudo memtester 1G 5

# 或使用memtest86+（需要重启到测试环境）
sudo apt install memtest86+
sudo update-grub
# 重启后在GRUB菜单选择memtest86+
```

## 11.5 存储设备

### 硬盘信息

```bash
# 列出所有块设备
lsblk
lsblk -f  # 显示文件系统

# 查看硬盘详细信息
sudo hdparm -I /dev/sda

# SMART信息（需smartmontools）
sudo apt install smartmontools

# 查看SMART状态
sudo smartctl -a /dev/sda

# 查看健康状态
sudo smartctl -H /dev/sda

# 运行短测试
sudo smartctl -t short /dev/sda

# 运行长测试
sudo smartctl -t long /dev/sda

# 查看测试结果
sudo smartctl -l selftest /dev/sda
```

### SSD优化

```bash
# 检查是否支持TRIM
sudo hdparm -I /dev/sda | grep TRIM

# 手动执行TRIM
sudo fstrim -v /

# 启用定期TRIM（Ubuntu默认已启用）
sudo systemctl enable fstrim.timer
sudo systemctl status fstrim.timer

# 查看SSD寿命
sudo smartctl -a /dev/sda | grep -i "wear"
```

### 硬盘性能测试

```bash
# hdparm读取测试
sudo hdparm -Tt /dev/sda

# dd写入测试
dd if=/dev/zero of=/tmp/test bs=1M count=1024 oflag=direct

# 使用fio（更专业）
sudo apt install fio

# 顺序读测试
fio --name=seqread --rw=read --bs=1M --size=1G --filename=/tmp/test

# 随机写测试
fio --name=randwrite --rw=randwrite --bs=4k --size=1G --filename=/tmp/test
```

## 11.6 显卡信息

### 查看显卡信息

```bash
# lspci查看显卡
lspci | grep -i vga
lspci | grep -i 3d

# 详细信息
lspci -v -s 00:02.0  # 替换为你的显卡ID

# 查看当前使用的显卡驱动
lspci -k | grep -A 3 -i vga
```

### NVIDIA显卡

```bash
# 查看NVIDIA驱动版本
nvidia-smi

# 实时监控
watch -n 1 nvidia-smi

# 查看GPU进程
nvidia-smi pmon

# 详细GPU信息
nvidia-smi -q

# 设置GPU风扇速度（需要X）
nvidia-settings
```

### AMD显卡

```bash
# 查看AMD GPU信息
lspci | grep -i amd

# 使用radeontop监控
sudo apt install radeontop
sudo radeontop
```

### Intel集成显卡

```bash
# 查看Intel GPU信息
sudo apt install intel-gpu-tools

# GPU频率
sudo intel_gpu_frequency

# GPU监控
sudo intel_gpu_top
```

## 11.7 网络硬件

### 网卡信息

```bash
# 查看网卡硬件
lspci | grep -i ethernet
lspci | grep -i network

# 查看网络接口
ip link show
ifconfig -a

# 网卡详细信息
ethtool eth0

# 查看网卡驱动
ethtool -i eth0

# 查看网卡统计
ethtool -S eth0

# 网卡速度和双工模式
ethtool eth0 | grep -i speed
ethtool eth0 | grep -i duplex
```

### 无线网卡

```bash
# 查看无线网卡
lspci | grep -i wireless
lsusb | grep -i wireless

# 无线接口信息
iwconfig

# 扫描WiFi
sudo iwlist wlan0 scan

# 无线统计
iwconfig wlan0 | grep -i quality
```

### 网络性能测试

```bash
# 安装iperf3
sudo apt install iperf3

# 服务端
iperf3 -s

# 客户端（测试到服务器）
iperf3 -c server_ip

# 测试带宽
iperf3 -c server_ip -t 30  # 测试30秒
```

## 11.8 电源管理

### 查看电源信息

```bash
# 笔记本电池信息
upower -i /org/freedesktop/UPower/devices/battery_BAT0

# 或使用acpi
sudo apt install acpi
acpi -V

# 电源统计
powertop  # 需要安装
```

### TLP - 电源优化

```bash
# 安装TLP（笔记本推荐）
sudo apt install tlp tlp-rdw

# 启动TLP
sudo systemctl enable tlp
sudo systemctl start tlp

# 查看TLP状态
sudo tlp-stat

# 查看电池信息
sudo tlp-stat -b
```

### 休眠和挂起

```bash
# 挂起（suspend）
systemctl suspend

# 休眠（hibernate）
systemctl hibernate

# 混合休眠
systemctl hybrid-sleep

# 检查是否支持休眠
cat /sys/power/state
```

## 11.9 温度和风扇

### 监控温度

```bash
# 安装lm-sensors
sudo apt install lm-sensors

# 检测传感器
sudo sensors-detect

# 查看温度
sensors

# 实时监控
watch -n 1 sensors

# 只显示温度
sensors | grep -i temp
```

### 风扇控制

```bash
# 安装fancontrol
sudo apt install fancontrol

# 配置风扇
sudo pwmconfig

# 启动fancontrol
sudo systemctl enable fancontrol
sudo systemctl start fancontrol
```

## 11.10 外设信息

### 输入设备

```bash
# 查看输入设备
cat /proc/bus/input/devices

# 鼠标键盘信息
xinput list  # 需要X环境

# 监控输入事件
evtest  # 需要安装
```

### 声卡

```bash
# 查看声卡
lspci | grep -i audio
aplay -l  # 列出播放设备
arecord -l  # 列出录音设备

# 音量控制
alsamixer

# 查看声音配置
cat /proc/asound/cards
```

### 打印机

```bash
# 查看USB打印机
lsusb | grep -i printer

# CUPS管理界面
# 浏览器访问 http://localhost:631

# 列出打印机
lpstat -p -d

# 查看打印队列
lpq
```

## 11.11 /proc和/sys文件系统

### /proc - 进程和系统信息

```bash
# CPU信息
cat /proc/cpuinfo

# 内存信息
cat /proc/meminfo

# 内核版本
cat /proc/version

# 负载
cat /proc/loadavg

# 运行时间
cat /proc/uptime

# 中断
cat /proc/interrupts

# IO统计
cat /proc/diskstats
```

### /sys - 硬件设备

```bash
# 块设备
ls /sys/block/

# 网络设备
ls /sys/class/net/

# 电源管理
ls /sys/class/power_supply/

# 背光
ls /sys/class/backlight/

# CPU频率
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
```

## 11.12 硬件故障排查

### 日志查看

```bash
# 内核日志
dmesg
dmesg | tail -50
dmesg | grep -i error

# 系统日志
journalctl -b  # 本次启动的日志
journalctl -p err  # 只看错误
journalctl -f  # 实时查看

# 硬件错误
journalctl -k | grep -i "hardware error"
```

### 内存错误

```bash
# 检查内存错误
sudo grep -i "memory" /var/log/syslog
sudo dmidecode -t memory | grep -i error

# 内存测试
sudo memtester 1G 1
```

### 硬盘错误

```bash
# 检查SMART错误
sudo smartctl -a /dev/sda | grep -i error

# 检查坏道
sudo badblocks -v /dev/sda

# 文件系统检查
sudo fsck /dev/sda1
```

### 硬件诊断工具

```bash
# 综合硬件测试
sudo apt install stress-ng
stress-ng --sequential 4 --timeout 60s

# CPU测试
stress-ng --cpu 4 --timeout 60s

# 内存测试
stress-ng --vm 2 --vm-bytes 2G --timeout 60s

# 磁盘测试
stress-ng --hdd 4 --timeout 60s
```

## 11.13 实用脚本

### 脚本1：硬件信息收集

```bash
#!/bin/bash
# hardware_info.sh - 收集硬件信息

OUTPUT="hardware_report.txt"

{
    echo "========== 系统信息 =========="
    uname -a
    echo ""

    echo "========== CPU信息 =========="
    lscpu | grep -E "Model name|CPU\(s\)|Thread|Core|Socket"
    echo ""

    echo "========== 内存信息 =========="
    free -h
    echo ""

    echo "========== 磁盘信息 =========="
    lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
    echo ""

    echo "========== 网卡信息 =========="
    ip -br link
    echo ""

    echo "========== PCI设备 =========="
    lspci | grep -E "VGA|Ethernet|Audio"
    echo ""

    echo "========== USB设备 =========="
    lsusb
    echo ""

    echo "========== 温度信息 =========="
    sensors 2>/dev/null || echo "lm-sensors未安装"

} > $OUTPUT

echo "硬件信息已保存到: $OUTPUT"
```

### 脚本2：硬件监控

```bash
#!/bin/bash
# hardware_monitor.sh - 实时硬件监控

while true; do
    clear
    echo "========== 硬件监控 =========="
    date
    echo ""

    echo "=== CPU使用率 ==="
    top -bn1 | grep "Cpu(s)" | awk '{print "CPU: " $2 " user, " $4 " sys"}'
    echo ""

    echo "=== 内存使用 ==="
    free -h | awk 'NR==2{printf "内存: %s / %s (%.2f%%)\n", $3, $2, $3/$2*100}'
    echo ""

    echo "=== 磁盘使用 ==="
    df -h / | awk 'NR==2{print "根目录: " $3 " / " $2 " (" $5 ")"}'
    echo ""

    echo "=== 温度 ==="
    sensors 2>/dev/null | grep -E "Core|temp" | head -5
    echo ""

    echo "按Ctrl+C退出"
    sleep 5
done
```

### 脚本3：硬件健康检查

```bash
#!/bin/bash
# hardware_health.sh - 硬件健康检查

echo "=== 硬件健康检查 ==="
echo ""

# 检查CPU温度
echo "检查CPU温度..."
TEMP=$(sensors 2>/dev/null | grep "Core 0" | awk '{print $3}' | tr -d '+°C')
if [ ! -z "$TEMP" ] && [ $(echo "$TEMP > 80" | bc) -eq 1 ]; then
    echo "警告: CPU温度过高 (${TEMP}°C)"
fi

# 检查内存使用
echo "检查内存使用..."
MEM_USAGE=$(free | awk 'NR==2{printf "%.0f", $3/$2*100}')
if [ $MEM_USAGE -gt 90 ]; then
    echo "警告: 内存使用率过高 (${MEM_USAGE}%)"
fi

# 检查磁盘使用
echo "检查磁盘使用..."
df -h | awk 'NR>1 && $5+0 > 90 {print "警告: " $6 " 使用率过高 (" $5 ")"}'

# 检查SMART状态
echo "检查磁盘健康..."
for disk in /dev/sd?; do
    if [ -e "$disk" ]; then
        HEALTH=$(sudo smartctl -H $disk 2>/dev/null | grep "PASSED")
        if [ -z "$HEALTH" ]; then
            echo "警告: $disk SMART状态异常"
        fi
    fi
done

echo ""
echo "检查完成！"
```

## 11.14 实践练习

### 练习1：查看系统硬件

```bash
# 1. 查看CPU信息
lscpu

# 2. 查看内存信息
free -h
sudo dmidecode -t memory

# 3. 查看磁盘信息
lsblk
sudo hdparm -I /dev/sda

# 4. 查看网卡信息
ip link
ethtool eth0
```

### 练习2：监控温度

```bash
# 安装并配置sensors
sudo apt install lm-sensors
sudo sensors-detect
sensors

# 实时监控
watch -n 1 sensors
```

### 练习3：硬件压力测试

```bash
# 安装stress
sudo apt install stress

# CPU测试
stress --cpu 4 --timeout 60s

# 同时监控温度
watch -n 1 sensors
```

## 下一步

完成本章后，你应该：
- ✅ 了解Linux硬件架构
- ✅ 掌握硬件信息查看命令
- ✅ 会监控硬件状态和温度
- ✅ 能够进行硬件故障排查
- ✅ 理解/proc和/sys文件系统

**准备好了吗？让我们进入[第12章：驱动管理](../12-驱动管理/README.md)！**

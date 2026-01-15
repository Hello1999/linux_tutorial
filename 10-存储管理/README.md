# 第10章：存储管理

## 10.1 磁盘基础知识

### 磁盘类型

**机械硬盘(HDD)**：
- 容量大、价格低
- 速度慢（~100MB/s）
- 有机械部件，怕震动
- 适合大容量存储

**固态硬盘(SSD)**：
- 速度快（500-3500MB/s）
- 无机械部件，抗震
- 价格较高
- 适合系统盘

**NVMe SSD**：
- 超快速度（3000-7000MB/s）
- 直接连接PCIe总线
- 最高性能
- 适合高性能需求

### 磁盘接口

```bash
# SATA设备：/dev/sda, /dev/sdb
# NVMe设备：/dev/nvme0n1, /dev/nvme0n2
# 旧IDE设备：/dev/hda, /dev/hdb
# 虚拟磁盘：/dev/vda, /dev/vdb（KVM）
```

### 分区表类型

**MBR (Master Boot Record)**：
- 传统分区表
- 最多4个主分区
- 最大支持2TB
- BIOS启动

**GPT (GUID Partition Table)**：
- 现代分区表
- 最多128个分区
- 支持超大磁盘（9.4ZB）
- UEFI启动
- 有备份分区表

## 10.2 查看磁盘信息

### lsblk - 列出块设备

```bash
# 查看所有块设备
lsblk

# 显示更多信息
lsblk -f    # 显示文件系统信息
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,FSTYPE

# 示例输出：
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0  500G  0 disk
# ├─sda1   8:1    0  512M  0 part /boot/efi
# ├─sda2   8:2    0  100G  0 part /
# └─sda3   8:3    0  399G  0 part /home
```

### fdisk - 查看分区信息

```bash
# 列出所有磁盘
sudo fdisk -l

# 查看特定磁盘
sudo fdisk -l /dev/sda

# 显示分区大小
sudo fdisk -l | grep "Disk /dev"
```

### df - 磁盘使用情况

```bash
# 查看磁盘使用情况
df -h           # 人类可读格式
df -h /         # 查看根分区
df -i           # 查看inode使用情况
df -T           # 显示文件系统类型

# 只显示本地文件系统
df -h -l

# 示例输出：
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda2       100G   45G   50G  48% /
# /dev/sda3       399G  120G  259G  32% /home
```

### du - 目录磁盘使用

```bash
# 查看目录大小
du -h /home/user              # 查看目录大小
du -sh /home/user             # 只显示总计（summary）
du -h --max-depth=1 /home     # 只显示1级深度

# 查找大文件
du -ah /home | sort -rh | head -20

# 排除某些目录
du -h --exclude="*.log" /var

# 查看当前目录各子目录大小
du -h --max-depth=1 | sort -hr
```

### 其他查看命令

```bash
# blkid - 查看UUID和文件系统类型
sudo blkid
sudo blkid /dev/sda1

# lshw - 硬件信息
sudo lshw -class disk

# smartctl - SMART健康信息（需安装smartmontools）
sudo smartctl -a /dev/sda

# hdparm - 硬盘参数
sudo hdparm -I /dev/sda
```

## 10.3 磁盘分区

### fdisk - MBR分区工具

```bash
# 启动fdisk
sudo fdisk /dev/sdb

# fdisk交互命令：
m       # 帮助
p       # 打印分区表
n       # 创建新分区
d       # 删除分区
t       # 改变分区类型
w       # 写入并退出
q       # 不保存退出

# 创建分区示例：
sudo fdisk /dev/sdb
# 输入: n → p → 1 → Enter → +10G → w
```

**完整分区示例**：

```bash
sudo fdisk /dev/sdb
# 1. 输入 p 查看当前分区
# 2. 输入 n 创建新分区
# 3. 选择 p (主分区) 或 e (扩展分区)
# 4. 输入分区号 (1-4)
# 5. 输入起始扇区 (默认回车)
# 6. 输入结束扇区 (+10G 表示10GB)
# 7. 输入 w 保存并退出
```

### parted - GPT分区工具

```bash
# 启动parted
sudo parted /dev/sdb

# parted交互命令：
print           # 显示分区表
mklabel gpt     # 创建GPT分区表
mklabel msdos   # 创建MBR分区表
mkpart          # 创建分区
rm              # 删除分区
quit            # 退出

# 创建GPT分区示例：
sudo parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 10GB
(parted) print
(parted) quit

# 非交互式创建分区：
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 0% 100%
```

### gdisk - GPT专用工具

```bash
# 启动gdisk
sudo gdisk /dev/sdb

# 命令类似fdisk
# n - 新建分区
# d - 删除分区
# p - 打印分区表
# w - 写入并退出
```

## 10.4 文件系统

### Linux文件系统类型

**ext4**（默认）：
- 最常用的Linux文件系统
- 支持日志
- 最大16TB文件
- 稳定可靠

**XFS**：
- 高性能文件系统
- 适合大文件
- RHEL/CentOS默认
- 不能缩小

**Btrfs**：
- 现代写时复制文件系统
- 支持快照、压缩
- 仍在发展中

**F2FS**：
- 专为闪存优化
- 适合SSD/eMMC

**FAT32/exFAT**：
- Windows兼容
- U盘常用

**NTFS**：
- Windows文件系统
- Linux可读写（需ntfs-3g）

### 创建文件系统

```bash
# ext4
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 -L MyDisk /dev/sdb1  # 带卷标

# XFS
sudo mkfs.xfs /dev/sdb1

# Btrfs
sudo mkfs.btrfs /dev/sdb1

# FAT32
sudo mkfs.vfat /dev/sdb1
sudo mkfs.vfat -F 32 /dev/sdb1

# exFAT (需安装exfat-utils)
sudo mkfs.exfat /dev/sdb1

# 交换分区
sudo mkswap /dev/sdb2
sudo swapon /dev/sdb2
```

### 检查和修复文件系统

```bash
# 检查ext4文件系统（需先卸载）
sudo umount /dev/sdb1
sudo fsck /dev/sdb1
sudo fsck.ext4 -f /dev/sdb1  # 强制检查

# 检查XFS
sudo xfs_repair /dev/sdb1

# 检查NTFS
sudo ntfsfix /dev/sdb1

# 自动修复错误
sudo fsck -y /dev/sdb1
```

## 10.5 挂载和卸载

### 手动挂载

```bash
# 创建挂载点
sudo mkdir /mnt/mydisk

# 挂载分区
sudo mount /dev/sdb1 /mnt/mydisk

# 指定文件系统类型
sudo mount -t ext4 /dev/sdb1 /mnt/mydisk

# 只读挂载
sudo mount -o ro /dev/sdb1 /mnt/mydisk

# 挂载ISO镜像
sudo mount -o loop ubuntu.iso /mnt/iso

# 挂载NFS
sudo mount -t nfs server:/share /mnt/nfs

# 挂载CIFS/SMB（Windows共享）
sudo mount -t cifs //server/share /mnt/smb -o username=user,password=pass
```

### 卸载

```bash
# 卸载
sudo umount /mnt/mydisk

# 强制卸载（谨慎使用）
sudo umount -f /mnt/mydisk

# 延迟卸载（等待进程结束）
sudo umount -l /mnt/mydisk

# 查看谁在使用
lsof /mnt/mydisk
fuser -m /mnt/mydisk

# 杀死使用进程
sudo fuser -km /mnt/mydisk
```

### 自动挂载 - /etc/fstab

`/etc/fstab` 配置开机自动挂载。

**fstab格式**：

```
设备       挂载点    类型  选项        dump  fsck
/dev/sdb1  /data     ext4  defaults    0     2
```

**字段说明**：
1. **设备**：分区路径、UUID或LABEL
2. **挂载点**：挂载目录
3. **类型**：文件系统类型
4. **选项**：挂载选项
5. **dump**：是否备份（0=否，1=是）
6. **fsck**：启动时检查顺序（0=不检查，1=首先检查，2=之后检查）

**常用挂载选项**：

```
defaults    # 默认选项（rw, suid, dev, exec, auto, nouser, async）
ro          # 只读
rw          # 读写
noatime     # 不更新访问时间（提高性能）
nodiratime  # 不更新目录访问时间
noexec      # 不允许执行程序
nosuid      # 忽略SUID位
nodev       # 不解析设备文件
auto        # 开机自动挂载
noauto      # 不自动挂载
user        # 允许普通用户挂载
nouser      # 只允许root挂载
```

**示例配置**：

```bash
sudo nano /etc/fstab

# 使用设备路径
/dev/sdb1  /data  ext4  defaults  0  2

# 使用UUID（推荐）
UUID=abc-123  /data  ext4  defaults  0  2

# 使用LABEL
LABEL=MyDisk  /data  ext4  defaults  0  2

# 交换分区
UUID=def-456  none  swap  sw  0  0

# NTFS分区
/dev/sdb2  /windows  ntfs-3g  defaults,uid=1000,gid=1000  0  0

# NFS挂载
server:/share  /mnt/nfs  nfs  defaults  0  0

# tmpfs（内存文件系统）
tmpfs  /tmp  tmpfs  defaults,noatime,mode=1777  0  0
```

**查看UUID**：

```bash
sudo blkid
# 或
lsblk -f
```

**测试fstab配置**：

```bash
# 卸载所有
sudo umount -a

# 重新挂载所有
sudo mount -a

# 或只测试特定挂载点
sudo mount /data
```

## 10.6 LVM (逻辑卷管理)

LVM提供灵活的磁盘管理，可以动态调整分区大小。

### LVM概念

```
物理卷(PV) → 卷组(VG) → 逻辑卷(LV) → 文件系统

/dev/sdb1 ──┐
            ├─→ VG1 ──→ LV1 (/data)
/dev/sdc1 ──┘       └─→ LV2 (/backup)
```

### 创建LVM

```bash
# 1. 创建物理卷(PV)
sudo pvcreate /dev/sdb1
sudo pvcreate /dev/sdc1

# 查看PV
sudo pvdisplay
sudo pvs

# 2. 创建卷组(VG)
sudo vgcreate my_vg /dev/sdb1 /dev/sdc1

# 查看VG
sudo vgdisplay
sudo vgs

# 3. 创建逻辑卷(LV)
sudo lvcreate -L 50G -n my_lv my_vg      # 创建50GB
sudo lvcreate -l 100%FREE -n my_lv my_vg # 使用所有空间

# 查看LV
sudo lvdisplay
sudo lvs

# 4. 创建文件系统
sudo mkfs.ext4 /dev/my_vg/my_lv

# 5. 挂载
sudo mkdir /mnt/mylv
sudo mount /dev/my_vg/my_lv /mnt/mylv
```

### 调整LVM大小

```bash
# 扩展逻辑卷
sudo lvextend -L +10G /dev/my_vg/my_lv
# 扩展文件系统
sudo resize2fs /dev/my_vg/my_lv    # ext4
sudo xfs_growfs /mnt/mylv           # XFS

# 缩小逻辑卷（ext4，需先卸载）
sudo umount /mnt/mylv
sudo e2fsck -f /dev/my_vg/my_lv
sudo resize2fs /dev/my_vg/my_lv 40G
sudo lvreduce -L 40G /dev/my_vg/my_lv
sudo mount /dev/my_vg/my_lv /mnt/mylv

# 扩展卷组
sudo vgextend my_vg /dev/sdd1
```

### LVM快照

```bash
# 创建快照
sudo lvcreate -L 5G -s -n my_lv_snap /dev/my_vg/my_lv

# 挂载快照
sudo mkdir /mnt/snapshot
sudo mount /dev/my_vg/my_lv_snap /mnt/snapshot

# 恢复快照
sudo umount /mnt/mylv
sudo lvconvert --merge /dev/my_vg/my_lv_snap
sudo mount /dev/my_vg/my_lv /mnt/mylv

# 删除快照
sudo lvremove /dev/my_vg/my_lv_snap
```

## 10.7 RAID

RAID (Redundant Array of Independent Disks) 提供数据冗余和性能提升。

### RAID级别

**RAID 0（条带化）**：
- 数据分散到多个磁盘
- 性能提升
- 无冗余（一块坏全完）

**RAID 1（镜像）**：
- 数据完全复制
- 容错能力强
- 可用空间减半

**RAID 5（带奇偶校验的条带化）**：
- 至少3块盘
- 允许1块盘损坏
- 性能和容量平衡

**RAID 6（双奇偶校验）**：
- 至少4块盘
- 允许2块盘损坏
- 更高容错

**RAID 10（1+0）**：
- 至少4块盘
- 镜像+条带
- 高性能+高可靠

### 软件RAID (mdadm)

```bash
# 安装mdadm
sudo apt install mdadm

# 创建RAID 1
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# 创建RAID 5
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# 查看RAID状态
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# 创建文件系统
sudo mkfs.ext4 /dev/md0

# 挂载
sudo mkdir /mnt/raid
sudo mount /dev/md0 /mnt/raid

# 保存RAID配置
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u

# 添加fstab
echo "/dev/md0 /mnt/raid ext4 defaults 0 2" | sudo tee -a /etc/fstab

# 模拟磁盘故障
sudo mdadm --manage /dev/md0 --fail /dev/sdb

# 移除故障盘
sudo mdadm --manage /dev/md0 --remove /dev/sdb

# 添加新盘
sudo mdadm --manage /dev/md0 --add /dev/sde

# 停止RAID
sudo mdadm --stop /dev/md0

# 删除RAID
sudo mdadm --zero-superblock /dev/sdb /dev/sdc
```

## 10.8 交换空间(Swap)

### 查看交换空间

```bash
# 查看swap使用情况
free -h
swapon --show
cat /proc/swaps
```

### 创建swap文件

```bash
# 1. 创建swap文件
sudo dd if=/dev/zero of=/swapfile bs=1G count=4  # 4GB
# 或使用fallocate（更快）
sudo fallocate -l 4G /swapfile

# 2. 设置权限
sudo chmod 600 /swapfile

# 3. 格式化为swap
sudo mkswap /swapfile

# 4. 启用swap
sudo swapon /swapfile

# 5. 永久启用（添加到fstab）
echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab

# 验证
swapon --show
```

### 调整swap优先级

```bash
# 查看swappiness（0-100，默认60）
cat /proc/sys/vm/swappiness

# 临时修改
sudo sysctl vm.swappiness=10

# 永久修改
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
```

### 删除swap

```bash
# 关闭swap
sudo swapoff /swapfile

# 从fstab删除
sudo nano /etc/fstab
# 删除swapfile行

# 删除文件
sudo rm /swapfile
```

## 10.9 磁盘配额

限制用户或组的磁盘使用。

```bash
# 1. 安装quota工具
sudo apt install quota

# 2. 编辑fstab，添加配额选项
/dev/sdb1 /data ext4 defaults,usrquota,grpquota 0 2

# 3. 重新挂载
sudo mount -o remount /data

# 4. 创建配额文件
sudo quotacheck -cugm /data

# 5. 启用配额
sudo quotaon /data

# 6. 为用户设置配额
sudo edquota -u username
# 或使用命令
sudo setquota -u username 1000000 1200000 0 0 /data
# 软限制1GB，硬限制1.2GB

# 7. 查看配额
quota -u username
sudo repquota /data

# 8. 为组设置配额
sudo edquota -g groupname

# 关闭配额
sudo quotaoff /data
```

## 10.10 磁盘性能测试

```bash
# hdparm测试读取速度
sudo hdparm -Tt /dev/sda

# dd测试写入速度
dd if=/dev/zero of=/tmp/test bs=1G count=1 oflag=direct

# fio综合测试（需安装）
sudo apt install fio

# 顺序读测试
fio --name=seqread --rw=read --bs=1M --size=1G --filename=/tmp/test

# 随机读写测试
fio --name=randwrite --rw=randwrite --bs=4k --size=1G --filename=/tmp/test

# iops测试
fio --name=randread --rw=randread --bs=4k --size=1G --iodepth=64 --filename=/tmp/test
```

## 10.11 实用脚本

### 脚本1：磁盘空间监控

```bash
#!/bin/bash
# disk_monitor.sh - 磁盘空间监控

THRESHOLD=80

df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{print $5 " " $1 " " $6}' | while read output;
do
    usage=$(echo $output | awk '{print $1}' | sed 's/%//g')
    partition=$(echo $output | awk '{print $2}')
    mount=$(echo $output | awk '{print $3}')

    if [ $usage -ge $THRESHOLD ]; then
        echo "警告: $partition ($mount) 使用率 ${usage}%"
        # 可以添加邮件通知或其他告警
    fi
done
```

### 脚本2：清理大文件

```bash
#!/bin/bash
# cleanup_large_files.sh - 查找并清理大文件

echo "查找大于100MB的文件..."
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | \
    awk '{print $5, $9}' | \
    sort -hr | \
    head -20
```

### 脚本3：自动备份脚本

```bash
#!/bin/bash
# backup.sh - 自动备份

SOURCE="/home/user/data"
DEST="/mnt/backup"
DATE=$(date +%Y%m%d)

# 创建备份
tar -czf "$DEST/backup_$DATE.tar.gz" "$SOURCE"

# 删除7天前的备份
find "$DEST" -name "backup_*.tar.gz" -mtime +7 -delete

echo "备份完成: backup_$DATE.tar.gz"
```

## 10.12 实践练习

### 练习1：分区和格式化

```bash
# 1. 列出所有磁盘
lsblk

# 2. 创建新分区（假设/dev/sdb）
sudo fdisk /dev/sdb
# 创建一个10GB分区

# 3. 格式化为ext4
sudo mkfs.ext4 /dev/sdb1

# 4. 挂载
sudo mkdir /mnt/test
sudo mount /dev/sdb1 /mnt/test

# 5. 测试
df -h /mnt/test
```

### 练习2：创建swap文件

```bash
# 创建2GB swap
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
free -h
```

### 练习3：LVM创建和调整

```bash
# 创建LVM（需要空闲分区）
sudo pvcreate /dev/sdb1
sudo vgcreate test_vg /dev/sdb1
sudo lvcreate -L 5G -n test_lv test_vg
sudo mkfs.ext4 /dev/test_vg/test_lv
sudo mkdir /mnt/lvm_test
sudo mount /dev/test_vg/test_lv /mnt/lvm_test

# 扩展到10G
sudo lvextend -L 10G /dev/test_vg/test_lv
sudo resize2fs /dev/test_vg/test_lv
```

## 下一步

完成本章后，你应该：
- ✅ 理解Linux存储架构
- ✅ 掌握磁盘分区和格式化
- ✅ 会使用LVM管理逻辑卷
- ✅ 了解RAID和swap
- ✅ 能够监控和优化磁盘使用

**准备好了吗？让我们进入[第11章：硬件知识](../11-硬件知识/README.md)！**

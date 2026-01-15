# 第2章：文件系统结构

## 2.1 一切皆文件

在Linux中，**一切皆文件**是最核心的设计哲学：
- 普通文件是文件
- 目录是文件
- 硬件设备是文件（在/dev下）
- 进程信息是文件（在/proc下）
- 网络套接字是文件

这种统一的抽象让操作变得简单一致。

## 2.2 Linux目录树结构

Linux使用**单一目录树**结构，所有文件系统都挂载到同一棵树上，根目录是 `/`

```
/                           (根目录 - 一切的起点)
├── bin/                    (基本命令二进制文件)
├── boot/                   (启动文件，包含内核)
├── dev/                    (设备文件)
├── etc/                    (系统配置文件)
├── home/                   (用户主目录)
│   ├── user1/
│   └── user2/
├── lib/                    (共享库文件)
├── media/                  (可移动媒体挂载点)
├── mnt/                    (临时挂载点)
├── opt/                    (可选应用软件包)
├── proc/                   (进程和内核信息)
├── root/                   (root用户的主目录)
├── run/                    (运行时数据)
├── sbin/                   (系统管理命令)
├── srv/                    (服务数据)
├── sys/                    (系统设备和驱动信息)
├── tmp/                    (临时文件)
├── usr/                    (用户程序)
│   ├── bin/
│   ├── lib/
│   ├── local/
│   └── share/
└── var/                    (可变数据)
    ├── log/                (日志文件)
    ├── cache/
    └── tmp/
```

### 对比Windows：
**Windows**：多个根（C:\, D:\, E:\）
**Linux**：单一根（/），其他分区挂载到目录上

## 2.3 重要目录详解

### `/` - 根目录
整个文件系统的顶层，所有路径的起点。

### `/bin` - 基本命令
存放所有用户都可以使用的基本命令
```bash
ls, cp, mv, rm, cat, chmod, bash, grep, tar
```
现代发行版中，`/bin`通常是指向`/usr/bin`的符号链接

### `/sbin` - 系统管理命令
系统管理员使用的命令
```bash
iptables, fdisk, mkfs, reboot, shutdown
```

### `/boot` - 启动文件
包含Linux启动所需的文件
```
/boot/
├── vmlinuz-*              (Linux内核)
├── initrd.img-*           (初始RAM磁盘)
├── grub/                  (GRUB引导加载程序配置)
└── config-*               (内核编译配置)
```

**重要**：不要随意删除这个目录的文件！

### `/dev` - 设备文件
Linux中硬件设备被抽象为文件

**块设备**（存储设备）：
```bash
/dev/sda          # 第一块SATA硬盘
/dev/sda1         # 第一块硬盘的第一个分区
/dev/nvme0n1      # NVMe SSD
/dev/sr0          # 光驱
```

**字符设备**：
```bash
/dev/tty          # 当前终端
/dev/null         # 数据黑洞（写入的数据全部丢弃）
/dev/zero         # 产生无限的零字节
/dev/random       # 随机数生成器
```

**伪设备**：
```bash
/dev/stdin        # 标准输入
/dev/stdout       # 标准输出
/dev/stderr       # 标准错误
```

### `/etc` - 配置文件
系统和应用程序的配置文件（Editable Text Configuration）

```bash
/etc/passwd            # 用户账户信息
/etc/shadow            # 用户密码（加密）
/etc/group             # 用户组信息
/etc/fstab             # 文件系统挂载配置
/etc/hosts             # 本地DNS解析
/etc/hostname          # 主机名
/etc/network/          # 网络配置
/etc/ssh/              # SSH配置
/etc/apt/              # APT包管理器配置（Debian/Ubuntu）
/etc/systemd/          # systemd配置
```

**命名规范**：配置文件通常以`.conf`结尾

### `/home` - 用户主目录
每个普通用户都有自己的主目录

```bash
/home/zhang/           # 用户zhang的主目录
/home/li/              # 用户li的主目录
```

用户在自己的主目录有完全控制权，可以用`~`表示：
```bash
cd ~              # 回到自己的主目录
cd ~/Documents    # 等同于 /home/当前用户/Documents
```

### `/root` - root用户主目录
超级用户root的主目录（不在/home下）

### `/lib` 和 `/lib64` - 共享库
系统启动和运行命令所需的共享库文件（类似Windows的DLL）

```bash
/lib/x86_64-linux-gnu/libc.so.6    # C标准库
/lib/modules/                       # 内核模块
```

### `/media` - 可移动媒体
自动挂载的可移动设备（U盘、光盘等）

```bash
/media/username/USB_DISK/
/media/username/CDROM/
```

### `/mnt` - 临时挂载点
手动挂载文件系统的临时目录

```bash
sudo mount /dev/sdb1 /mnt
```

### `/opt` - 可选软件包
第三方软件通常安装在这里

```bash
/opt/google/chrome/
/opt/teamviewer/
```

### `/proc` - 进程信息（虚拟文件系统）
内核和进程信息的接口，内容不存储在磁盘上，而是由内核动态生成

```bash
/proc/cpuinfo          # CPU信息
/proc/meminfo          # 内存信息
/proc/version          # 内核版本
/proc/[PID]/           # 特定进程的信息
/proc/[PID]/cmdline    # 进程启动命令
/proc/[PID]/status     # 进程状态
```

**示例**：查看CPU信息
```bash
cat /proc/cpuinfo
```

### `/run` - 运行时数据
系统启动以来的运行时数据（每次启动清空）

```bash
/run/user/1000/        # 用户运行时数据
/run/lock/             # 锁文件
```

### `/sys` - 系统设备信息（虚拟文件系统）
内核导出的设备和驱动信息

```bash
/sys/class/net/        # 网络接口
/sys/block/            # 块设备
/sys/devices/          # 设备树
```

### `/tmp` - 临时文件
所有用户都可以写入的临时文件目录

**特点**：
- 系统重启后内容会被清空（或定期清理）
- 权限：`drwxrwxrwt`（粘滞位，只有所有者可以删除自己的文件）

```bash
# 临时解压文件
tar -xzf archive.tar.gz -C /tmp
```

### `/usr` - 用户程序
**不是**"user"的缩写，而是"Unix System Resources"

```
/usr/
├── bin/               # 用户命令
├── sbin/              # 系统管理命令
├── lib/               # 库文件
├── local/             # 本地安装的软件
│   ├── bin/
│   ├── lib/
│   └── share/
├── share/             # 共享数据
│   ├── doc/           # 文档
│   ├── man/           # 手册页
│   └── applications/  # 桌面快捷方式
└── src/               # 源代码
```

**安装软件的位置**：
- 系统包管理器安装：`/usr/bin`
- 手动编译安装：`/usr/local/bin`

### `/var` - 可变数据
经常变化的文件

```bash
/var/log/              # 日志文件
/var/cache/            # 应用缓存
/var/tmp/              # 临时文件（重启后保留）
/var/spool/            # 队列数据（打印、邮件）
/var/lib/              # 应用数据（数据库文件等）
/var/www/              # Web服务器文件
```

**重要日志**：
```bash
/var/log/syslog        # 系统日志（Debian/Ubuntu）
/var/log/messages      # 系统日志（RHEL/CentOS）
/var/log/auth.log      # 认证日志
/var/log/kern.log      # 内核日志
/var/log/dmesg         # 启动消息
```

## 2.4 文件类型

Linux有7种文件类型：

| 符号 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `-` | 普通文件 | 文本、二进制、图片等 | `-rw-r--r-- file.txt` |
| `d` | 目录 | 文件夹 | `drwxr-xr-x Documents/` |
| `l` | 符号链接 | 快捷方式 | `lrwxrwxrwx link -> target` |
| `c` | 字符设备 | 字符流设备 | `crw------- /dev/tty` |
| `b` | 块设备 | 块存储设备 | `brw-rw---- /dev/sda` |
| `s` | 套接字 | 网络或IPC通信 | `srwxrwxrwx /run/docker.sock` |
| `p` | 管道 | 命名管道 | `prw------- mypipe` |

查看文件类型：
```bash
ls -l /dev/sda
# brw-rw---- 1 root disk 8, 0 Jan  7 10:00 /dev/sda
# ↑ b表示块设备

file /bin/ls
# /bin/ls: ELF 64-bit LSB executable
```

## 2.5 路径

### 绝对路径
从根目录`/`开始的完整路径
```bash
/home/zhang/Documents/file.txt
/etc/nginx/nginx.conf
```

### 相对路径
相对于当前目录的路径
```bash
./file.txt         # 当前目录下的file.txt
../file.txt        # 上级目录下的file.txt
Documents/file.txt # 当前目录下Documents目录中的file.txt
```

### 特殊路径符号
```bash
.       # 当前目录
..      # 父目录
~       # 当前用户主目录
~user   # 指定用户的主目录
-       # 上一次所在的目录
```

**示例**：
```bash
cd /etc
cd ~           # 回到主目录 /home/zhang
cd -           # 回到上一次的目录 /etc
```

## 2.6 文件系统类型

### 常见文件系统：

**Linux原生**：
- **ext4** - 第四代扩展文件系统，最常用
- **XFS** - 高性能日志文件系统
- **Btrfs** - B-tree文件系统，支持快照、压缩
- **ZFS** - 超强大的文件系统（非Linux原生，需单独安装）

**Windows**：
- **NTFS** - Linux可以读写（需ntfs-3g）
- **FAT32** - 兼容性最好，但有4GB文件大小限制
- **exFAT** - FAT32的改进版

**网络**：
- **NFS** - 网络文件系统
- **CIFS/SMB** - Windows网络共享

**虚拟**：
- **tmpfs** - 内存文件系统（/dev/shm）
- **proc** - 进程文件系统
- **sysfs** - 系统设备文件系统

查看文件系统类型：
```bash
df -T
# 或
lsblk -f
```

## 2.7 挂载（Mount）

在Linux中，使用存储设备前需要将其**挂载**到目录树的某个位置。

### 查看挂载情况
```bash
mount              # 显示所有挂载点
df -h              # 显示磁盘使用情况
lsblk              # 树状显示块设备
```

### 手动挂载
```bash
# 挂载U盘
sudo mount /dev/sdb1 /mnt

# 挂载ISO文件
sudo mount -o loop image.iso /mnt

# 挂载Windows分区（NTFS）
sudo mount -t ntfs-3g /dev/sda1 /mnt/windows
```

### 卸载
```bash
sudo umount /mnt
# 或
sudo umount /dev/sdb1
```

### 自动挂载：/etc/fstab
系统启动时自动挂载的配置文件

格式：
```
设备    挂载点    类型    选项    dump    fsck
```

示例 `/etc/fstab`：
```bash
# <file system>  <mount point>  <type>  <options>       <dump>  <pass>
UUID=xxx-xxx     /              ext4    defaults        0       1
UUID=yyy-yyy     /home          ext4    defaults        0       2
UUID=zzz-zzz     swap           swap    defaults        0       0
/dev/sdb1        /mnt/data      ext4    defaults        0       2
```

**重要**：编辑/etc/fstab前务必备份，错误配置可能导致系统无法启动！

### 查找设备UUID
```bash
sudo blkid
# 或
lsblk -o NAME,UUID
```

## 2.8 磁盘空间管理

### 查看磁盘使用情况
```bash
# 查看分区使用情况
df -h              # human-readable格式
df -i              # 查看inode使用情况

# 查看目录大小
du -sh /var/log    # 显示总大小
du -h --max-depth=1 /var  # 显示一级子目录大小
```

### 查找大文件
```bash
# 查找大于100MB的文件
find / -type f -size +100M 2>/dev/null

# 查找并排序
du -sh /* 2>/dev/null | sort -hr | head -10
```

### 清理空间
```bash
# 清理APT缓存（Debian/Ubuntu）
sudo apt clean
sudo apt autoclean

# 清理日志
sudo journalctl --vacuum-time=7d  # 保留最近7天

# 清理临时文件
rm -rf ~/.cache/*
```

## 2.9 符号链接与硬链接

### 符号链接（软链接）
类似Windows的快捷方式

```bash
# 创建符号链接
ln -s /path/to/target linkname

# 示例：
ln -s /var/www/html ~/website
# 现在 ~/website 指向 /var/www/html
```

特点：
- 可以跨文件系统
- 可以链接目录
- 目标删除后，链接失效（成为悬空链接）

### 硬链接
文件的另一个入口，与原文件共享inode

```bash
# 创建硬链接
ln /path/to/file hardlink
```

特点：
- 不能跨文件系统
- 不能链接目录
- 任何一个链接删除，文件仍然存在（直到所有链接都删除）

### 对比
```bash
# 创建测试文件
echo "test" > original.txt

# 创建符号链接
ln -s original.txt soft_link.txt

# 创建硬链接
ln original.txt hard_link.txt

# 查看
ls -li
# 会显示inode编号，硬链接的inode相同
```

## 2.10 文件权限初步

使用`ls -l`查看文件权限：
```bash
$ ls -l /etc/passwd
-rw-r--r-- 1 root root 2841 Jan  5 10:23 /etc/passwd
```

解读：
```
-rw-r--r--  1  root  root  2841  Jan  5 10:23  /etc/passwd
│││││││││  │   │     │     │      │           │
│││││││││  │   │     │     │      │           └─ 文件名
│││││││││  │   │     │     │      └─ 修改时间
│││││││││  │   │     │     └─ 大小（字节）
│││││││││  │   │     └─ 所属组
│││││││││  │   └─ 所有者
│││││││││  └─ 硬链接数
│└┴┴└┴┴└┴┴─ 权限
└─ 文件类型
```

权限详解将在[第4章：用户和权限管理](../04-用户权限/README.md)深入讲解。

## 2.11 实践练习

### 练习1：探索目录结构
```bash
# 查看根目录
ls -l /

# 查看home目录
ls -l /home

# 查看当前用户主目录
ls -la ~

# 查看系统日志目录
ls -l /var/log
```

### 练习2：查看系统信息
```bash
# CPU信息
cat /proc/cpuinfo | grep "model name" | head -1

# 内存信息
cat /proc/meminfo | grep MemTotal

# 内核版本
cat /proc/version
```

### 练习3：磁盘管理
```bash
# 查看分区
lsblk

# 查看磁盘使用
df -h

# 查看当前目录大小
du -sh .

# 查看主目录下各目录大小
du -h --max-depth=1 ~
```

### 练习4：创建符号链接
```bash
# 在主目录创建测试
cd ~
mkdir test_links
cd test_links

# 创建文件
echo "Hello Linux" > original.txt

# 创建符号链接
ln -s original.txt soft_link.txt

# 创建硬链接
ln original.txt hard_link.txt

# 查看
ls -li
cat soft_link.txt
cat hard_link.txt

# 删除原文件
rm original.txt

# 再次查看链接
cat soft_link.txt   # 会报错（符号链接失效）
cat hard_link.txt   # 仍然可以访问（硬链接有效）
```

### 练习5：挂载实验（安全操作）
```bash
# 查看当前挂载
mount | grep "^/dev"

# 查看块设备
lsblk

# 查看文件系统类型
df -T
```

## 2.12 常见问题

### Q: /usr/bin 和 /usr/local/bin 有什么区别？
**A**:
- `/usr/bin` - 系统包管理器安装的软件
- `/usr/local/bin` - 手动编译安装的软件（不受包管理器管理）

### Q: 为什么我的U盘在/media下自动挂载？
**A**: 现代桌面环境使用`udisks2`自动挂载可移动设备到`/media/用户名/`

### Q: /tmp 和 /var/tmp 有什么区别？
**A**:
- `/tmp` - 重启后清空
- `/var/tmp` - 重启后保留，但会定期清理旧文件

### Q: 什么时候用绝对路径，什么时候用相对路径？
**A**:
- **脚本中**：尽量用绝对路径（避免因工作目录改变出错）
- **命令行临时操作**：相对路径更方便
- **配置文件**：通常需要绝对路径

## 下一步

完成本章后，你应该：
- ✅ 理解Linux目录树结构
- ✅ 知道重要目录的作用
- ✅ 掌握路径表示方法
- ✅ 了解挂载概念
- ✅ 会基本的磁盘空间查看

**准备好了吗？让我们进入[第3章：基本命令入门](../03-基本命令/README.md)，开始实际操作！**

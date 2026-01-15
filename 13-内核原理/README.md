# 第13章：Linux内核原理

## 13.1 内核概述

### 什么是内核？

内核是操作系统的核心，负责：
- 进程管理
- 内存管理
- 文件系统
- 设备驱动
- 网络协议栈
- 系统调用接口

### Linux内核架构

```
┌────────────────────────────────────────┐
│          用户空间（User Space）          │
│     应用程序、Shell、系统工具等          │
└────────────────────────────────────────┘
              ↕ 系统调用接口
┌────────────────────────────────────────┐
│         内核空间（Kernel Space）         │
│ ┌────────────────────────────────────┐ │
│ │        进程调度和进程管理           │ │
│ ├────────────────────────────────────┤ │
│ │          内存管理（MM）            │ │
│ ├────────────────────────────────────┤ │
│ │      虚拟文件系统（VFS）           │ │
│ ├────────────────────────────────────┤ │
│ │         网络协议栈（Net）          │ │
│ ├────────────────────────────────────┤ │
│ │        设备驱动程序                │ │
│ └────────────────────────────────────┘ │
│        进程间通信（IPC）               │
└────────────────────────────────────────┘
              ↕ 硬件抽象层
┌────────────────────────────────────────┐
│            硬件（Hardware）             │
│   CPU、内存、磁盘、网卡、外设等         │
└────────────────────────────────────────┘
```

### 内核版本

```bash
# 查看内核版本
uname -r
# 输出示例：6.5.0-14-generic
# 6 - 主版本号
# 5 - 次版本号（偶数=稳定版，奇数=开发版）
# 0 - 修订版本号
# 14 - 发行版补丁版本

# 查看详细信息
cat /proc/version
uname -a

# 查看内核编译时间
uname -v
```

### 内核类型

**单内核（Monolithic Kernel）**：
- Linux使用的类型
- 所有服务在内核空间运行
- 性能高，但不够灵活

**微内核（Microkernel）**：
- 只有最基本功能在内核空间
- 其他服务在用户空间
- 更安全，但性能较低
- 例如：Minix、QNX

**混合内核（Hybrid Kernel）**：
- 结合两者优点
- 例如：Windows NT

## 13.2 系统调用

### 什么是系统调用？

系统调用是用户程序访问内核服务的接口。

### 常见系统调用

```c
// 进程管理
fork()      // 创建子进程
exec()      // 执行新程序
exit()      // 终止进程
wait()      // 等待子进程
getpid()    // 获取进程ID

// 文件操作
open()      // 打开文件
read()      // 读文件
write()     // 写文件
close()     // 关闭文件
lseek()     // 移动文件指针

// 内存管理
brk()       // 改变数据段大小
mmap()      // 内存映射
munmap()    // 取消映射

// 信号处理
signal()    // 注册信号处理函数
kill()      // 发送信号
```

### 查看系统调用

```bash
# 使用strace跟踪系统调用
strace ls

# 只显示特定系统调用
strace -e open ls
strace -e open,read,write ls

# 跟踪正在运行的进程
strace -p PID

# 统计系统调用
strace -c ls

# 跟踪子进程
strace -f command
```

## 13.3 进程管理

### 进程状态

```
创建 → 就绪 → 运行 → 终止
        ↕      ↕
       等待   挂起
```

**进程状态**：
- R (Running): 运行或就绪
- S (Sleeping): 可中断睡眠
- D (Disk sleep): 不可中断睡眠（通常是IO）
- T (Stopped): 停止
- Z (Zombie): 僵尸进程
- X (Dead): 已终止

### 进程结构

```c
// task_struct（进程控制块）
struct task_struct {
    pid_t pid;               // 进程ID
    pid_t tgid;              // 线程组ID
    struct task_struct *parent;  // 父进程
    struct list_head children;   // 子进程列表
    struct mm_struct *mm;    // 内存描述符
    struct files_struct *files;  // 文件描述符
    ...
};
```

### 进程调度

**调度算法**：

1. **CFS (Completely Fair Scheduler)**：
   - Linux默认调度器
   - 完全公平调度
   - 基于虚拟运行时间

2. **实时调度器**：
   - SCHED_FIFO: 先进先出
   - SCHED_RR: 时间片轮转

### 查看进程信息

```bash
# 进程树
pstree
pstree -p  # 显示PID

# 进程详细信息
ps aux
ps -ef

# 进程状态
cat /proc/PID/status

# 进程内存映射
cat /proc/PID/maps

# 进程打开的文件
lsof -p PID

# 进程命令行
cat /proc/PID/cmdline

# 进程环境变量
cat /proc/PID/environ
```

### 进程优先级

```bash
# nice值：-20（最高优先级）到 19（最低优先级）
# 启动时设置nice值
nice -n 10 command

# 调整运行中进程的nice值
renice -n 5 -p PID

# 查看进程优先级
ps -el | grep PID
top  # 按r键调整nice值
```

## 13.4 内存管理

### 虚拟内存

Linux使用虚拟内存技术：
- 每个进程拥有独立的地址空间（32位系统：4GB）
- 虚拟地址通过页表映射到物理地址
- 支持内存保护和共享

### 内存布局

```
高地址
┌──────────────┐
│   内核空间   │  1GB (x86_32)
├──────────────┤
│     栈       │  ↓ 向下增长
├──────────────┤
│      ↓       │
│   (空闲)     │
│      ↑       │
├──────────────┤
│     堆       │  ↑ 向上增长
├──────────────┤
│   BSS段      │  未初始化数据
├──────────────┤
│   数据段     │  已初始化数据
├──────────────┤
│   代码段     │  程序代码
└──────────────┘
低地址
```

### 页面管理

- **页大小**：通常为4KB
- **页表**：虚拟地址→物理地址映射
- **TLB**：Translation Lookaside Buffer，快速地址转换缓存

```bash
# 查看页大小
getconf PAGE_SIZE

# 查看内存页统计
cat /proc/vmstat

# 查看内存碎片
cat /proc/buddyinfo
```

### 交换空间

```bash
# 查看swap
cat /proc/swaps
swapon --show

# Swappiness（0-100，控制使用swap的倾向）
cat /proc/sys/vm/swappiness
# 0 = 尽量不用swap
# 100 = 积极使用swap

# 设置swappiness
sudo sysctl vm.swappiness=10
```

### OOM Killer

当内存耗尽时，OOM Killer会杀死进程。

```bash
# 查看OOM日志
dmesg | grep -i oom
journalctl -b | grep -i oom

# OOM得分（越高越容易被杀）
cat /proc/PID/oom_score

# 设置OOM优先级（-1000到1000）
echo -17 > /proc/PID/oom_adj  # 几乎不会被杀
echo 0 > /proc/PID/oom_adj    # 默认
echo 15 > /proc/PID/oom_adj   # 更容易被杀
```

## 13.5 文件系统

### VFS (虚拟文件系统)

VFS提供统一的文件系统接口，支持多种文件系统类型。

```
应用程序
    ↓
 系统调用
    ↓
   VFS层
    ↓
┌────┬────┬────┐
│ext4│XFS │Btrfs│ ...
└────┴────┴────┘
    ↓
 块设备层
    ↓
 磁盘驱动
```

### 文件系统结构

```c
// 超级块（Superblock）
struct super_block {
    unsigned long s_blocksize;  // 块大小
    unsigned long s_maxbytes;   // 最大文件大小
    struct file_system_type *s_type;
    ...
};

// inode（索引节点）
struct inode {
    umode_t i_mode;     // 文件类型和权限
    uid_t i_uid;        // 所有者
    gid_t i_gid;        // 组
    loff_t i_size;      // 文件大小
    time_t i_atime;     // 访问时间
    time_t i_mtime;     // 修改时间
    time_t i_ctime;     // 状态改变时间
    ...
};
```

### inode

```bash
# 查看inode信息
ls -i file.txt
stat file.txt

# 查看inode使用情况
df -i

# 查找inode号对应的文件
find / -inum 12345

# 删除无法删除的文件（通过inode）
find / -inum 12345 -delete
```

### 文件描述符

```bash
# 查看进程打开的文件
lsof -p PID
ls -l /proc/PID/fd

# 查看系统文件描述符限制
ulimit -n
cat /proc/sys/fs/file-max

# 设置限制
ulimit -n 4096
# 永久设置：编辑 /etc/security/limits.conf
```

### 缓存

```bash
# 查看缓存
free -h
cat /proc/meminfo | grep -i cache

# 清理缓存（需root）
sync  # 同步磁盘
echo 1 > /proc/sys/vm/drop_caches  # 清理页缓存
echo 2 > /proc/sys/vm/drop_caches  # 清理dentries和inodes
echo 3 > /proc/sys/vm/drop_caches  # 清理所有缓存
```

## 13.6 进程间通信（IPC）

### IPC机制

1. **管道（Pipe）**：
```bash
command1 | command2
```

2. **命名管道（FIFO）**：
```bash
mkfifo /tmp/mypipe
echo "hello" > /tmp/mypipe &
cat /tmp/mypipe
```

3. **信号（Signal）**：
```bash
kill -SIGUSR1 PID
killall -HUP process_name
```

4. **共享内存**：
```bash
# 查看共享内存
ipcs -m
# 删除共享内存
ipcrm -m shmid
```

5. **消息队列**：
```bash
# 查看消息队列
ipcs -q
```

6. **信号量**：
```bash
# 查看信号量
ipcs -s
```

7. **Socket**：
网络通信或本地IPC

## 13.7 网络协议栈

### 网络层次

```
应用层    (HTTP, FTP, SSH...)
传输层    (TCP, UDP)
网络层    (IP, ICMP, ARP)
链路层    (Ethernet, WiFi)
物理层    (硬件)
```

### 网络统计

```bash
# 查看网络连接
netstat -tuln
ss -tuln  # 更快的替代品

# 查看路由表
ip route
route -n

# 查看ARP缓存
ip neigh
arp -n

# 网络统计
netstat -s
ss -s

# 查看网络接口统计
ip -s link
```

### 网络参数调优

```bash
# TCP参数
cat /proc/sys/net/ipv4/tcp_*

# 示例：调整TCP缓冲区
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 6291456"
sudo sysctl -w net.ipv4.tcp_wmem="4096 16384 4194304"

# 启用TCP快速打开
sudo sysctl -w net.ipv4.tcp_fastopen=3

# 调整连接队列
sudo sysctl -w net.core.somaxconn=4096
```

## 13.8 内核参数调优

### sysctl - 内核参数管理

```bash
# 查看所有内核参数
sysctl -a

# 查看特定参数
sysctl vm.swappiness

# 临时修改
sudo sysctl -w vm.swappiness=10

# 永久修改
sudo nano /etc/sysctl.conf
# 添加：vm.swappiness=10
sudo sysctl -p  # 应用配置
```

### 常用内核参数

```bash
# 虚拟内存
vm.swappiness=10              # Swap使用倾向
vm.dirty_ratio=15             # 脏页比例
vm.dirty_background_ratio=5   # 后台写回比例

# 文件系统
fs.file-max=2097152           # 最大文件描述符
fs.inotify.max_user_watches=524288  # inotify监视数

# 网络
net.core.somaxconn=4096       # 连接队列长度
net.ipv4.tcp_max_syn_backlog=8192  # SYN队列长度
net.ipv4.ip_local_port_range=1024 65535  # 本地端口范围

# 内核
kernel.pid_max=4194304        # 最大进程数
kernel.threads-max=4194304    # 最大线程数
```

## 13.9 内核模块开发

### 简单内核模块

```c
// hello.c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A simple Hello World module");

static int __init hello_init(void) {
    printk(KERN_INFO "Hello, Kernel!\n");
    return 0;
}

static void __exit hello_exit(void) {
    printk(KERN_INFO "Goodbye, Kernel!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

### Makefile

```makefile
obj-m += hello.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

### 编译和加载

```bash
# 编译
make

# 加载模块
sudo insmod hello.ko

# 查看内核日志
dmesg | tail

# 卸载模块
sudo rmmod hello

# 查看模块信息
modinfo hello.ko
```

## 13.10 内核编译

### 获取内核源码

```bash
# 方法1：包管理器
sudo apt install linux-source
cd /usr/src
sudo tar -xjf linux-source-*.tar.bz2

# 方法2：kernel.org
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.5.tar.xz
tar -xf linux-6.5.tar.xz
cd linux-6.5
```

### 编译内核

```bash
# 1. 安装编译工具
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev

# 2. 配置内核
make menuconfig  # 图形化配置
# 或
cp /boot/config-$(uname -r) .config  # 使用当前配置
make oldconfig

# 3. 编译（使用所有CPU核心）
make -j$(nproc)

# 4. 编译模块
make modules -j$(nproc)

# 5. 安装模块
sudo make modules_install

# 6. 安装内核
sudo make install

# 7. 更新GRUB
sudo update-grub

# 8. 重启
sudo reboot
```

### 自定义内核

```bash
# 只编译需要的驱动
make localmodconfig  # 基于当前加载的模块

# 清理
make clean        # 删除编译文件
make mrproper     # 删除所有生成文件
make distclean    # 完全清理
```

## 13.11 内核调试

### printk调试

```c
printk(KERN_DEBUG "Debug message\n");
printk(KERN_INFO "Info message\n");
printk(KERN_WARNING "Warning message\n");
printk(KERN_ERR "Error message\n");
```

```bash
# 查看内核日志
dmesg
dmesg -w  # 实时查看

# 设置日志级别
dmesg -n 8  # 显示所有消息
```

### kprobes - 动态探测

```bash
# 安装perf
sudo apt install linux-tools-generic

# 列出可探测点
sudo perf probe -F

# 添加探测点
sudo perf probe --add="do_sys_open filename:string"

# 查看事件
sudo perf probe -l

# 删除探测点
sudo perf probe --del=probe:do_sys_open
```

### crash分析

```bash
# 安装crash工具
sudo apt install crash

# 分析内核转储
crash /usr/lib/debug/boot/vmlinux-$(uname -r) /var/crash/...
```

## 13.12 实用工具

### perf - 性能分析

```bash
# 系统范围性能统计
sudo perf stat command

# CPU采样
sudo perf record -a -g sleep 10
sudo perf report

# 实时top
sudo perf top
```

### ftrace - 内核跟踪

```bash
# 启用ftrace
sudo mount -t debugfs none /sys/kernel/debug
cd /sys/kernel/debug/tracing

# 查看可用跟踪器
cat available_tracers

# 启用函数跟踪
echo function > current_tracer

# 查看跟踪结果
cat trace

# 停止跟踪
echo nop > current_tracer
```

## 13.13 实践练习

### 练习1：系统调用跟踪

```bash
# 跟踪ls命令的系统调用
strace ls

# 只显示文件操作
strace -e open,read,close ls

# 统计系统调用
strace -c ls
```

### 练习2：内存分析

```bash
# 查看进程内存
ps aux --sort=-%mem | head -10

# 查看内存详情
cat /proc/meminfo

# 查看进程内存映射
cat /proc/PID/maps
```

### 练习3：编写简单内核模块

```bash
# 创建hello模块（参考上面的代码）
# 编译、加载、测试、卸载
```

## 下一步

完成本章后，你应该：
- ✅ 理解Linux内核架构
- ✅ 了解进程和内存管理
- ✅ 掌握内核参数调优
- ✅ 能够编译内核和模块
- ✅ 会使用内核调试工具

**准备好了吗？让我们进入[第14章：性能优化](../14-性能优化/README.md)！**

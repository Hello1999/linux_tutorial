# 第14章：Linux性能优化

## 14.1 性能优化概述

### 性能优化方法论

```
1. 测量 → 2. 分析 → 3. 优化 → 4. 验证
     ↑_____________________________↓
```

**优化原则**：
- 先测量再优化（避免过早优化）
- 找出瓶颈（80/20法则）
- 一次优化一个方面
- 验证优化效果

### 性能指标

**系统性能指标**：
- CPU使用率
- 内存使用率
- 磁盘I/O
- 网络带宽
- 响应时间
- 吞吐量

## 14.2 CPU性能优化

### CPU监控

```bash
# 实时CPU监控
top
htop  # 更友好

# CPU详细统计
mpstat 1  # 每秒更新
sar -u 1  # 需安装sysstat

# 每个CPU核心
mpstat -P ALL 1

# CPU使用历史
sar -u

# 查看CPU瓶颈
uptime  # 查看负载
```

### 负载理解

```bash
# 查看系统负载
uptime
# 15:30:15 up 10 days, 3:45, 2 users, load average: 0.15, 0.20, 0.18
#                                                    1分钟 5分钟 15分钟

# 负载含义（假设4核CPU）：
# < 4.0  : 正常
# = 4.0  : 满载
# > 4.0  : 等待队列

# 查看CPU核心数
nproc
```

### CPU优化技巧

```bash
# 1. 进程优先级调整
nice -n 10 command      # 启动时设置
renice -n 5 -p PID      # 调整运行中的进程

# 2. CPU亲和性（绑定进程到特定CPU）
taskset -c 0,1 command  # 绑定到CPU 0和1
taskset -p 0x3 PID      # 二进制掩码（0x3=CPU 0,1）

# 3. 限制CPU使用
cpulimit -p PID -l 50   # 限制到50%

# 4. CPU调度策略
chrt -f 99 command      # FIFO实时调度
chrt -r 50 command      # RR实时调度
```

### 查找CPU消耗进程

```bash
# Top 10 CPU消耗进程
ps aux --sort=-%cpu | head -11

# 实时监控
top -o %CPU

# 查看线程
top -H
ps -eLf
```

## 14.3 内存性能优化

### 内存监控

```bash
# 基本内存信息
free -h

# 详细内存统计
vmstat 1

# 内存使用详情
cat /proc/meminfo

# 进程内存排序
ps aux --sort=-%mem | head -11

# 查看进程内存
pmap -x PID
cat /proc/PID/smaps
```

### 内存优化

```bash
# 1. 调整swappiness
cat /proc/sys/vm/swappiness
sudo sysctl vm.swappiness=10
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf

# 2. 透明大页
cat /sys/kernel/mm/transparent_hugepage/enabled
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/enabled

# 3. 清理缓存（不推荐常用）
sync
echo 3 | sudo tee /proc/sys/vm/drop_caches

# 4. 限制进程内存（cgroup）
sudo systemd-run --scope -p MemoryLimit=1G command
```

### 内存泄漏检测

```bash
# 使用valgrind
sudo apt install valgrind
valgrind --leak-check=full ./program

# 监控内存增长
while true; do
    ps aux | grep program_name
    sleep 5
done

# 查看内存映射
pmap PID
cat /proc/PID/maps
```

## 14.4 磁盘I/O优化

### 磁盘监控

```bash
# iostat（需安装sysstat）
iostat -x 1

# iotop（需安装）
sudo iotop

# 查看磁盘使用
df -h
du -sh /*

# 磁盘I/O统计
cat /proc/diskstats
```

### I/O调度器

```bash
# 查看当前调度器
cat /sys/block/sda/queue/scheduler

# 可用调度器
# [mq-deadline] none
# mq-deadline: 多队列截止时间调度器（默认）
# none: 无调度器（适合NVMe SSD）

# 修改调度器
echo none | sudo tee /sys/block/sda/queue/scheduler

# 永久修改（/etc/udev/rules.d/60-scheduler.rules）
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/scheduler}="mq-deadline"
ACTION=="add|change", KERNEL=="nvme*", ATTR{queue/scheduler}="none"
```

### 文件系统优化

```bash
# 1. 挂载选项优化
# /etc/fstab
UUID=xxx / ext4 defaults,noatime,nodiratime 0 1

# noatime: 不更新访问时间（提升性能）
# nodiratime: 不更新目录访问时间
# commit=60: 延迟60秒写入（默认5秒）

# 2. 文件系统调优
# ext4
sudo tune2fs -o journal_data_writeback /dev/sda1

# XFS
sudo xfs_io -c "resblks percent=5" /mount/point

# 3. 预读优化
sudo blockdev --setra 8192 /dev/sda
```

### 磁盘性能测试

```bash
# hdparm测试
sudo hdparm -Tt /dev/sda

# dd测试写入
dd if=/dev/zero of=/tmp/test bs=1M count=1024 oflag=direct

# fio综合测试
sudo apt install fio

# 顺序读
fio --name=seqread --rw=read --bs=1M --size=1G --filename=/tmp/test

# 随机写
fio --name=randwrite --rw=randwrite --bs=4k --size=1G --iodepth=64 --filename=/tmp/test

# IOPS测试
fio --name=iops --rw=randread --bs=4k --runtime=60 --time_based --iodepth=32 --filename=/dev/sda
```

## 14.5 网络性能优化

### 网络监控

```bash
# 实时网络监控
iftop      # 需安装
nload      # 需安装
nethogs    # 按进程显示

# 网络统计
netstat -i
ip -s link

# 连接统计
netstat -ant | awk '{print $6}' | sort | uniq -c

# 网络带宽测试
iperf3 -s   # 服务端
iperf3 -c server_ip  # 客户端
```

### TCP优化

```bash
# /etc/sysctl.conf

# TCP缓冲区
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 16384 16777216

# TCP连接
net.core.somaxconn=4096
net.ipv4.tcp_max_syn_backlog=8192
net.ipv4.tcp_max_tw_buckets=5000

# TCP快速打开
net.ipv4.tcp_fastopen=3

# TIME_WAIT复用
net.ipv4.tcp_tw_reuse=1

# TCP拥塞控制
net.ipv4.tcp_congestion_control=bbr

# 应用修改
sudo sysctl -p
```

### 网络性能测试

```bash
# 延迟测试
ping -c 10 server

# 带宽测试
iperf3 -c server -t 30

# 并发连接测试
ab -n 10000 -c 100 http://server/
wrk -t4 -c100 -d30s http://server/
```

## 14.6 综合性能分析工具

### top - 系统监控

```bash
top

# 常用快捷键：
# P - 按CPU排序
# M - 按内存排序
# k - 杀死进程
# r - 调整nice值
# 1 - 显示所有CPU核心
# c - 显示完整命令
```

### htop - 增强版top

```bash
sudo apt install htop
htop

# 特点：
# - 彩色显示
# - 鼠标支持
# - 树形进程视图
# - 更直观的操作
```

### atop - 全面监控

```bash
sudo apt install atop
atop

# 特点：
# - 记录历史数据
# - 显示磁盘I/O
# - 显示网络
# - 可以回放历史
```

### glances - 现代化监控

```bash
sudo apt install glances
glances

# Web模式
glances -w
# 访问 http://localhost:61208
```

### perf - 性能分析

```bash
# 安装
sudo apt install linux-tools-generic

# 系统范围统计
sudo perf stat sleep 10

# CPU采样
sudo perf record -a -g sleep 10
sudo perf report

# 实时top
sudo perf top

# 查看特定进程
sudo perf record -p PID -g sleep 10
```

### sar - 系统活动报告

```bash
# 安装
sudo apt install sysstat

# CPU使用
sar -u 1 10  # 每秒一次，共10次

# 内存
sar -r 1 10

# 磁盘I/O
sar -d 1 10

# 网络
sar -n DEV 1 10

# 查看历史数据
sar -u -f /var/log/sysstat/sa$(date +%d)
```

## 14.7 数据库性能优化

### MySQL/MariaDB

```bash
# 查看慢查询
sudo tail -f /var/log/mysql/mysql-slow.log

# 启用慢查询日志
# /etc/mysql/my.cnf
[mysqld]
slow_query_log=1
slow_query_log_file=/var/log/mysql/mysql-slow.log
long_query_time=2

# 查看状态
mysql -u root -p -e "SHOW PROCESSLIST;"
mysql -u root -p -e "SHOW STATUS;"

# 分析查询
EXPLAIN SELECT * FROM table WHERE condition;
```

### PostgreSQL

```bash
# 查看慢查询
# /etc/postgresql/*/main/postgresql.conf
log_min_duration_statement=1000  # 记录超过1秒的查询

# 查看活动查询
psql -c "SELECT * FROM pg_stat_activity;"

# 分析查询
EXPLAIN ANALYZE SELECT * FROM table WHERE condition;
```

## 14.8 Web服务器优化

### Nginx优化

```nginx
# /etc/nginx/nginx.conf

# Worker进程数（通常等于CPU核心数）
worker_processes auto;

# 每个worker的连接数
events {
    worker_connections 1024;
    use epoll;  # Linux高效事件模型
}

http {
    # 启用gzip压缩
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript;

    # 文件缓存
    open_file_cache max=1000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;

    # 客户端缓存
    location ~* \.(jpg|jpeg|png|gif|css|js)$ {
        expires 30d;
    }

    # 限流
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
    limit_req zone=one burst=20;
}
```

### Apache优化

```apache
# /etc/apache2/apache2.conf

# MPM配置（多进程模块）
<IfModule mpm_prefork_module>
    StartServers          5
    MinSpareServers       5
    MaxSpareServers      10
    MaxRequestWorkers   150
    MaxConnectionsPerChild 3000
</IfModule>

# 启用压缩
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css
    AddOutputFilterByType DEFLATE application/javascript application/json
</IfModule>

# 启用缓存
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpg "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 month"
    ExpiresByType image/png "access plus 1 month"
    ExpiresByType text/css "access plus 1 week"
    ExpiresByType application/javascript "access plus 1 week"
</IfModule>
```

## 14.9 应用程序优化

### 编译优化

```bash
# GCC优化级别
gcc -O0  # 无优化（调试）
gcc -O1  # 基本优化
gcc -O2  # 推荐优化级别
gcc -O3  # 积极优化
gcc -Os  # 优化大小

# 针对本地CPU优化
gcc -march=native -O2 program.c

# 链接时优化
gcc -flto -O2 program.c
```

### Python优化

```bash
# 使用PyPy（JIT编译器）
sudo apt install pypy3
pypy3 script.py

# 使用Cython编译
pip install cython
cython script.py
gcc -shared -pthread -fPIC -fwrapv -O2 -Wall -fno-strict-aliasing \
    -I/usr/include/python3.x -o script.so script.c

# 使用多进程
from multiprocessing import Pool

# 使用缓存
from functools import lru_cache
@lru_cache(maxsize=128)
def expensive_function(arg):
    ...
```

## 14.10 实用优化脚本

### 脚本1：系统性能检查

```bash
#!/bin/bash
# performance_check.sh

echo "=== 系统性能检查 ==="
date
echo ""

# CPU
echo "=== CPU负载 ==="
uptime
echo ""

# 内存
echo "=== 内存使用 ==="
free -h
echo ""

# 磁盘
echo "=== 磁盘使用 ==="
df -h
echo ""

# 磁盘I/O
echo "=== 磁盘I/O ==="
iostat -x 1 5
echo ""

# 网络
echo "=== 网络连接 ==="
netstat -an | awk '/^tcp/ {print $6}' | sort | uniq -c
echo ""

# Top进程
echo "=== Top 10 CPU进程 ==="
ps aux --sort=-%cpu | head -11
echo ""

echo "=== Top 10 内存进程 ==="
ps aux --sort=-%mem | head -11
```

### 脚本2：自动优化

```bash
#!/bin/bash
# auto_optimize.sh

echo "开始系统优化..."

# 清理缓存
echo "清理包缓存..."
sudo apt autoremove -y
sudo apt clean

# 清理日志
echo "清理旧日志..."
sudo journalctl --vacuum-time=7d

# 优化swappiness
echo "优化swap使用..."
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 启用TCP BBR
echo "启用TCP BBR..."
echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 磁盘优化
echo "优化磁盘I/O..."
for disk in /sys/block/sd?/queue/scheduler; do
    echo mq-deadline | sudo tee $disk
done

echo "优化完成！"
```

### 脚本3：性能监控

```bash
#!/bin/bash
# monitor.sh

LOG="/var/log/performance.log"

while true; do
    {
        echo "=== $(date) ==="
        echo "CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}')"
        echo "内存: $(free | awk 'NR==2{printf "%.2f%%", $3*100/$2}')"
        echo "磁盘: $(df -h / | awk 'NR==2{print $5}')"
        echo ""
    } >> $LOG

    sleep 60
done
```

## 14.11 性能基准测试

### UnixBench

```bash
# 安装依赖
sudo apt install build-essential

# 下载
git clone https://github.com/kdlucas/byte-unixbench.git
cd byte-unixbench/UnixBench

# 运行测试
./Run

# 结果在 results/ 目录
```

### sysbench

```bash
# 安装
sudo apt install sysbench

# CPU测试
sysbench cpu --cpu-max-prime=20000 run

# 内存测试
sysbench memory --memory-total-size=10G run

# 文件I/O测试
sysbench fileio --file-total-size=10G prepare
sysbench fileio --file-total-size=10G --file-test-mode=rndrw run
sysbench fileio --file-total-size=10G cleanup
```

## 14.12 实践练习

### 练习1：识别性能瓶颈

```bash
# 1. 查看系统负载
uptime

# 2. 查找CPU消耗最大的进程
ps aux --sort=-%cpu | head -5

# 3. 查看内存使用
free -h

# 4. 查看磁盘I/O
iostat -x 1 5

# 5. 查看网络连接
netstat -ant | wc -l
```

### 练习2：优化系统参数

```bash
# 优化swappiness
sudo sysctl vm.swappiness=10

# 优化TCP参数
sudo sysctl net.ipv4.tcp_rmem="4096 87380 16777216"
sudo sysctl net.ipv4.tcp_wmem="4096 16384 16777216"

# 验证
sysctl vm.swappiness
sysctl net.ipv4.tcp_rmem
```

### 练习3：性能测试

```bash
# 磁盘性能测试
sudo hdparm -Tt /dev/sda
dd if=/dev/zero of=/tmp/test bs=1M count=1024

# 网络性能测试（需两台机器）
# 机器A：iperf3 -s
# 机器B：iperf3 -c <机器A的IP>
```

## 下一步

完成本章后，你应该：
- ✅ 掌握系统性能监控方法
- ✅ 了解各类性能瓶颈
- ✅ 会优化CPU、内存、磁盘、网络
- ✅ 能够使用性能分析工具
- ✅ 掌握常见服务的优化方法

**准备好了吗？让我们进入[第15章：安全加固](../15-安全加固/README.md)！**

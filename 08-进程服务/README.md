# 第8章：进程和服务管理

## 8.1 进程基础

### 什么是进程？
进程是正在运行的程序实例，每个进程有独立的：
- 进程ID (PID)
- 内存空间
- 文件描述符
- 环境变量

### 进程状态
- **R** (Running) - 运行中
- **S** (Sleeping) - 睡眠
- **D** (Disk Sleep) - 不可中断睡眠
- **T** (Stopped) - 停止
- **Z** (Zombie) - 僵尸进程

## 8.2 查看进程

```bash
# 查看所有进程
ps aux
ps -ef

# 进程树
pstree
ps auxf

# 动态查看
top
htop

# 查找特定进程
ps aux | grep nginx
pgrep nginx
pidof nginx
```

## 8.3 进程管理

```bash
# 启动后台进程
command &

# 杀死进程
kill PID
kill -9 PID        # 强制杀死
killall nginx      # 按名字杀死
pkill -f pattern   # 按模式杀死

# 进程优先级
nice -n 10 command      # 以低优先级启动
renice -n 5 -p PID      # 调整优先级

# 后台任务管理
jobs                    # 查看后台任务
fg %1                   # 调到前台
bg %1                   # 继续后台运行
Ctrl+Z                  # 暂停当前任务
```

## 8.4 systemd服务管理

### 基本命令

```bash
# 启动/停止/重启服务
sudo systemctl start service_name
sudo systemctl stop service_name
sudo systemctl restart service_name
sudo systemctl reload service_name

# 查看状态
systemctl status service_name
systemctl is-active service_name
systemctl is-enabled service_name

# 开机启动
sudo systemctl enable service_name
sudo systemctl disable service_name

# 列出服务
systemctl list-units --type=service
systemctl list-unit-files --type=service
```

### 创建自定义服务

```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# 重新加载配置
sudo systemctl daemon-reload

# 启动服务
sudo systemctl start myapp
sudo systemctl enable myapp
```

## 8.5 日志管理

```bash
# 查看系统日志
journalctl

# 特定服务日志
journalctl -u nginx
journalctl -u nginx -f        # 实时跟踪
journalctl -u nginx -n 50     # 最后50行

# 按时间查看
journalctl --since "2026-01-01"
journalctl --since "1 hour ago"
journalctl --since today
journalctl --until "2026-01-07 12:00"

# 按优先级查看
journalctl -p err             # 只看错误
journalctl -p warning..err    # 警告到错误

# 清理日志
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=100M
```

## 8.6 定时任务

### cron

```bash
# 编辑crontab
crontab -e

# 格式：分 时 日 月 周 命令
# * * * * * command
# ┬ ┬ ┬ ┬ ┬
# │ │ │ │ │
# │ │ │ │ └─── 星期 (0-7, 0和7都表示周日)
# │ │ │ └───── 月份 (1-12)
# │ │ └─────── 日期 (1-31)
# │ └───────── 小时 (0-23)
# └─────────── 分钟 (0-59)

# 示例
0 2 * * * /usr/local/bin/backup.sh                # 每天2点
*/5 * * * * /usr/local/bin/monitor.sh             # 每5分钟
0 0 * * 0 /usr/local/bin/weekly-task.sh           # 每周日0点
0 3 1 * * /usr/local/bin/monthly-task.sh          # 每月1日3点

# 查看cron任务
crontab -l

# 查看系统cron日志
grep CRON /var/log/syslog
```

### systemd定时器

```bash
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
OnCalendar=02:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
# 启用定时器
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# 查看定时器
systemctl list-timers
```

## 8.7 进程监控

```bash
# top使用
top
# 快捷键：
# M - 按内存排序
# P - 按CPU排序
# k - 杀死进程
# r - 调整优先级
# 1 - 显示所有CPU核心

# htop（更好用）
htop

# 查看进程打开的文件
lsof -p PID
lsof /path/to/file

# 查看进程系统调用
strace -p PID
strace command
```

## 8.8 实战练习

### 练习1：监控Nginx

```bash
# 检查Nginx状态
systemctl status nginx

# 查看Nginx进程
ps aux | grep nginx

# 实时监控Nginx日志
sudo tail -f /var/log/nginx/access.log

# 重载配置
sudo nginx -t                    # 测试配置
sudo systemctl reload nginx      # 重载
```

### 练习2：创建监控脚本服务

```bash
# 1. 创建脚本
sudo nano /usr/local/bin/monitor.sh

#!/bin/bash
while true; do
    CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
    echo "[$(date)] CPU: $CPU%" >> /var/log/monitor.log
    sleep 60
done

# 2. 赋予执行权限
sudo chmod +x /usr/local/bin/monitor.sh

# 3. 创建service文件
sudo nano /etc/systemd/system/monitor.service

[Unit]
Description=System Monitor Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/monitor.sh
Restart=always

[Install]
WantedBy=multi-user.target

# 4. 启动服务
sudo systemctl daemon-reload
sudo systemctl start monitor
sudo systemctl enable monitor

# 5. 查看日志
tail -f /var/log/monitor.log
```

### 练习3：定时任务管理

创建一个每日备份任务：

```bash
# 1. 创建备份脚本
cat > /usr/local/bin/daily-backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d)

# 创建备份目录
mkdir -p "$BACKUP_DIR"

# 备份重要目录
tar -czf "$BACKUP_DIR/home_$DATE.tar.gz" /home
tar -czf "$BACKUP_DIR/etc_$DATE.tar.gz" /etc

# 删除30天前的备份
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "备份完成: $(date)" >> /var/log/backup.log
EOF

sudo chmod +x /usr/local/bin/daily-backup.sh

# 2. 使用cron设置定时任务（每天凌晨2点）
sudo crontab -e
# 添加：0 2 * * * /usr/local/bin/daily-backup.sh

# 或者使用systemd timer
# 创建 /etc/systemd/system/daily-backup.service
sudo nano /etc/systemd/system/daily-backup.service
```

service文件内容：
```ini
[Unit]
Description=Daily Backup Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/daily-backup.sh
```

创建timer文件：
```bash
sudo nano /etc/systemd/system/daily-backup.timer
```

timer文件内容：
```ini
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
OnCalendar=02:00
Persistent=true

[Install]
WantedBy=timers.target
```

启用timer：
```bash
sudo systemctl daemon-reload
sudo systemctl enable daily-backup.timer
sudo systemctl start daily-backup.timer

# 查看timer状态
systemctl list-timers
systemctl status daily-backup.timer
```

### 练习4：进程优先级管理

实践进程优先级调整：

```bash
# 1. 启动一个CPU密集型进程
# 创建测试脚本
cat > cpu-test.sh << 'EOF'
#!/bin/bash
while true; do
    echo "scale=10000; 4*a(1)" | bc -l > /dev/null
done
EOF
chmod +x cpu-test.sh

# 2. 以不同优先级启动进程
./cpu-test.sh &                  # 默认优先级
nice -n 10 ./cpu-test.sh &       # 低优先级
nice -n -10 ./cpu-test.sh &      # 高优先级（需要root）

# 3. 查看进程优先级
ps -eo pid,ni,comm | grep cpu-test

# 4. 使用top查看
top
# 按 'r' 键，输入PID，调整nice值

# 5. 使用renice调整运行中的进程
renice -n 5 -p PID

# 6. 清理测试进程
killall cpu-test.sh
```

### 练习5：日志分析实战

使用journalctl进行日志分析：

```bash
# 1. 查找最近的错误
journalctl -p err -n 50

# 2. 查看特定时间段的日志
journalctl --since "2026-01-10 10:00" --until "2026-01-10 12:00"

# 3. 查看特定服务的启动失败日志
journalctl -u nginx --since today | grep -i fail

# 4. 统计今天各服务的错误数量
journalctl --since today -p err | grep "systemd\[" | \
    awk '{print $5}' | sort | uniq -c | sort -rn

# 5. 导出日志到文件
journalctl -u nginx --since "1 week ago" > nginx_week.log

# 6. 清理旧日志（保留最近7天）
sudo journalctl --vacuum-time=7d

# 7. 限制日志大小（最多100MB）
sudo journalctl --vacuum-size=100M
```

### 练习6：服务依赖管理

创建相互依赖的服务：

```bash
# 场景：创建一个Web应用服务，依赖于数据库服务

# 1. 创建模拟数据库服务
sudo nano /etc/systemd/system/mydb.service
```

```ini
[Unit]
Description=My Database Service
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c 'while true; do sleep 1; done'
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
# 2. 创建Web应用服务（依赖数据库）
sudo nano /etc/systemd/system/webapp.service
```

```ini
[Unit]
Description=My Web Application
After=network.target mydb.service
Requires=mydb.service

[Service]
Type=simple
ExecStart=/bin/bash -c 'echo "WebApp started"; while true; do sleep 1; done'
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
# 3. 启动并测试依赖关系
sudo systemctl daemon-reload
sudo systemctl start webapp

# 查看依赖树
systemctl list-dependencies webapp

# 4. 测试：停止数据库服务，观察webapp的行为
sudo systemctl stop mydb
systemctl status webapp

# 5. 清理
sudo systemctl stop webapp mydb
sudo systemctl disable webapp mydb
```

### 练习7：进程故障排查

实践进程问题诊断：

```bash
# 1. 找出占用CPU最高的进程
ps aux --sort=-%cpu | head -10
top -o %CPU

# 2. 找出占用内存最高的进程
ps aux --sort=-%mem | head -10
top -o %MEM

# 3. 查看进程打开的文件
lsof -p PID
ls -l /proc/PID/fd

# 4. 查看进程的环境变量
cat /proc/PID/environ | tr '\0' '\n'

# 5. 查看进程的系统调用
strace -p PID

# 6. 查看进程的网络连接
lsof -i -a -p PID
netstat -anp | grep PID

# 7. 分析僵尸进程
ps aux | grep 'Z'
ps -ef | grep defunct
```

### 练习8：后台任务管理

实践后台任务控制：

```bash
# 1. 启动长时间运行的任务
sleep 300 &
SLEEP_PID=$!

# 2. 查看后台任务
jobs
jobs -l

# 3. 将后台任务调到前台
fg %1

# 4. 暂停当前任务（Ctrl+Z），然后继续在后台运行
# 按 Ctrl+Z 暂停
bg %1

# 5. 使用nohup在退出后继续运行
nohup long-running-script.sh &

# 6. 使用screen或tmux管理会话
screen -S mysession
# 运行任务
# 按 Ctrl+A, D 分离会话
screen -r mysession  # 重新连接

# 7. 查看所有用户的后台进程
ps aux | grep [用户名]
```

### 挑战练习：自动故障恢复系统

创建一个智能的服务监控和自动恢复系统：

```bash
#!/bin/bash
# 服务监控和自动恢复脚本
# /usr/local/bin/service-watchdog.sh

SERVICES=("nginx" "mysql" "redis")
MAX_RETRIES=3
LOG_FILE="/var/log/service-watchdog.log"

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

check_and_restart() {
    local service=$1
    local retry_count=0

    if ! systemctl is-active --quiet "$service"; then
        log_message "警告: $service 服务未运行"

        while [ $retry_count -lt $MAX_RETRIES ]; do
            log_message "尝试重启 $service (第 $((retry_count+1)) 次)"
            systemctl restart "$service"
            sleep 5

            if systemctl is-active --quiet "$service"; then
                log_message "成功: $service 已重启"
                return 0
            fi

            ((retry_count++))
        done

        log_message "错误: $service 重启失败，已达最大重试次数"
        # 发送告警（可以集成邮件或webhook）
        return 1
    fi

    return 0
}

# 主循环
while true; do
    for service in "${SERVICES[@]}"; do
        check_and_restart "$service"
    done
    sleep 60  # 每分钟检查一次
done
```

要求：
1. 将脚本配置为systemd服务
2. 添加邮件告警功能（使用mail命令）
3. 添加服务重启次数统计
4. 实现不同服务的不同监控策略
5. 添加配置文件支持

## 学习资源

- `man systemd`
- `man systemctl`
- `man journalctl`
- `man ps`
- `man top`
- systemd官方文档
- [ArchWiki - systemd](https://wiki.archlinux.org/title/Systemd)
- [Systemd for Administrators](https://www.freedesktop.org/wiki/Software/systemd/)

## 下一步

完成本章后，你应该：
- ✅ 理解进程的基本概念和状态
- ✅ 掌握systemd服务管理
- ✅ 会配置定时任务（cron和systemd timer）
- ✅ 能够使用journalctl查看和分析日志
- ✅ 掌握进程监控和故障排查技巧
- ✅ 能够创建和管理自定义服务

**下一章：[第9章：网络管理](../09-网络管理/README.md)**

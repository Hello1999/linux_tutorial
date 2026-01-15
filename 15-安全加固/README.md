# 第15章：Linux安全加固

## 15.1 安全概述

### 安全基本原则

1. **最小权限原则**：只授予必要的权限
2. **纵深防御**：多层安全措施
3. **默认拒绝**：不明确允许的都拒绝
4. **定期审计**：监控和日志审查
5. **及时更新**：保持系统和软件最新

### 安全威胁类型

- 未授权访问
- 权限提升
- 恶意软件
- DDoS攻击
- 数据泄露
- 中间人攻击

## 15.2 用户和权限安全

### 账户安全

```bash
# 1. 禁用root直接登录
sudo nano /etc/ssh/sshd_config
# 设置：PermitRootLogin no
sudo systemctl restart sshd

# 2. 设置密码策略
sudo nano /etc/login.defs
# PASS_MAX_DAYS 90      # 密码最长有效期
# PASS_MIN_DAYS 7       # 密码最短使用期
# PASS_MIN_LEN 12       # 最小密码长度
# PASS_WARN_AGE 14      # 过期前警告天数

# 3. 密码复杂度要求
sudo apt install libpam-pwquality
sudo nano /etc/pam.d/common-password
# 添加：
password requisite pam_pwquality.so retry=3 minlen=12 dcredit=-1 ucredit=-1 ocredit=-1 lcredit=-1

# 4. 锁定失败登录
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# 5. 查看登录失败
sudo lastb

# 6. 禁用不必要的用户
sudo usermod -L username  # 锁定账户
sudo userdel -r username  # 删除账户
```

### sudo配置

```bash
# 编辑sudoers
sudo visudo

# 只允许特定命令
username ALL=(ALL) /usr/bin/systemctl, /usr/bin/apt

# 不需要密码（谨慎）
username ALL=(ALL) NOPASSWD: /usr/bin/specific_command

# 限制sudo超时
Defaults timestamp_timeout=5  # 5分钟

# 记录sudo命令
Defaults log_year, log_host, logfile="/var/log/sudo.log"
```

### 文件权限

```bash
# 检查权限过大的文件
find / -type f -perm 777 2>/dev/null
find / -type d -perm 777 2>/dev/null

# 查找SUID文件
find / -perm /4000 2>/dev/null

# 查找SGID文件
find / -perm /2000 2>/dev/null

# 删除不必要的SUID位
sudo chmod u-s /path/to/file

# 重要文件权限
sudo chmod 600 /etc/shadow
sudo chmod 644 /etc/passwd
sudo chmod 600 /boot/grub/grub.cfg
sudo chmod 600 ~/.ssh/*
sudo chmod 644 ~/.ssh/*.pub
```

## 15.3 SSH安全

### SSH配置加固

```bash
# 编辑SSH配置
sudo nano /etc/ssh/sshd_config

# 推荐配置：
Port 2222                          # 改变默认端口
PermitRootLogin no                 # 禁止root登录
PasswordAuthentication no          # 只允许密钥认证
PubkeyAuthentication yes           # 启用公钥认证
PermitEmptyPasswords no            # 禁止空密码
X11Forwarding no                   # 禁用X11转发
MaxAuthTries 3                     # 最大尝试次数
ClientAliveInterval 300            # 5分钟超时
ClientAliveCountMax 2              # 超时计数
AllowUsers user1 user2             # 只允许特定用户
DenyUsers baduser                  # 拒绝特定用户
Protocol 2                         # 只使用SSH协议2

# 重启SSH服务
sudo systemctl restart sshd
```

### SSH密钥认证

```bash
# 生成密钥对
ssh-keygen -t ed25519 -C "your_email@example.com"
# 或RSA（更兼容）
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 复制公钥到服务器
ssh-copy-id user@server

# 或手动复制
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# 设置正确权限
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# 使用SSH Agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### SSH双因素认证

```bash
# 安装Google Authenticator
sudo apt install libpam-google-authenticator

# 配置PAM
sudo nano /etc/pam.d/sshd
# 添加：
auth required pam_google_authenticator.so

# 编辑SSH配置
sudo nano /etc/ssh/sshd_config
# 设置：
ChallengeResponseAuthentication yes

# 用户设置
google-authenticator
# 使用手机App扫描二维码

# 重启SSH
sudo systemctl restart sshd
```

## 15.4 防火墙配置

### UFW (Ubuntu防火墙)

```bash
# 安装UFW
sudo apt install ufw

# 默认策略
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 允许SSH（改变端口后修改）
sudo ufw allow 2222/tcp

# 允许HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 允许特定IP
sudo ufw allow from 192.168.1.100

# 允许特定IP访问特定端口
sudo ufw allow from 192.168.1.100 to any port 22

# 拒绝特定IP
sudo ufw deny from 192.168.1.200

# 查看规则
sudo ufw status numbered
sudo ufw status verbose

# 删除规则
sudo ufw delete 2  # 按编号删除

# 启用防火墙
sudo ufw enable

# 禁用防火墙
sudo ufw disable

# 重置所有规则
sudo ufw reset
```

### iptables

```bash
# 查看规则
sudo iptables -L -v -n
sudo iptables -L INPUT -v -n

# 保存规则
sudo iptables-save > /etc/iptables/rules.v4

# 恢复规则
sudo iptables-restore < /etc/iptables/rules.v4

# 基本规则示例
# 默认拒绝所有
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# 允许本地回环
sudo iptables -A INPUT -i lo -j ACCEPT

# 允许已建立的连接
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 允许SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 允许HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 防止SYN洪水攻击
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP

# 防止端口扫描
sudo iptables -N port-scanning
sudo iptables -A port-scanning -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s --limit-burst 2 -j RETURN
sudo iptables -A port-scanning -j DROP
```

## 15.5 入侵检测

### fail2ban

```bash
# 安装
sudo apt install fail2ban

# 配置
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# 基本配置：
[DEFAULT]
bantime = 3600        # 封禁1小时
findtime = 600        # 10分钟内
maxretry = 5          # 失败5次

[sshd]
enabled = true
port = 2222           # SSH端口
logpath = /var/log/auth.log
maxretry = 3

# 查看状态
sudo fail2ban-client status
sudo fail2ban-client status sshd

# 解封IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# 查看被封IP
sudo fail2ban-client status sshd | grep "Banned IP"
```

### AIDE - 文件完整性检查

```bash
# 安装
sudo apt install aide

# 初始化数据库
sudo aideinit

# 复制数据库
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# 检查
sudo aide --check

# 更新数据库
sudo aide --update
```

### rkhunter - Rootkit检测

```bash
# 安装
sudo apt install rkhunter

# 更新
sudo rkhunter --update

# 扫描
sudo rkhunter --check

# 配置自动扫描
sudo nano /etc/cron.daily/rkhunter
#!/bin/bash
/usr/bin/rkhunter --update --quiet
/usr/bin/rkhunter --check --skip-keypress --report-warnings-only
```

## 15.6 系统加固

### SELinux / AppArmor

```bash
# AppArmor（Ubuntu默认）
# 查看状态
sudo apparmor_status

# 查看配置文件
ls /etc/apparmor.d/

# 强制模式
sudo aa-enforce /etc/apparmor.d/usr.bin.nginx

# 投诉模式（仅记录）
sudo aa-complain /etc/apparmor.d/usr.bin.nginx

# 禁用profile
sudo aa-disable /etc/apparmor.d/usr.bin.nginx
```

### 内核加固

```bash
# /etc/sysctl.conf

# 禁用IP转发（非路由器）
net.ipv4.ip_forward=0

# 防止SYN洪水
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_max_syn_backlog=2048
net.ipv4.tcp_synack_retries=2

# 防止ICMP重定向
net.ipv4.conf.all.accept_redirects=0
net.ipv6.conf.all.accept_redirects=0
net.ipv4.conf.all.send_redirects=0

# 防止源路由
net.ipv4.conf.all.accept_source_route=0
net.ipv6.conf.all.accept_source_route=0

# 防止IP欺骗
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.default.rp_filter=1

# 忽略ICMP ping
net.ipv4.icmp_echo_ignore_all=1

# 记录可疑包
net.ipv4.conf.all.log_martians=1

# 应用配置
sudo sysctl -p
```

### 禁用不必要的服务

```bash
# 列出所有服务
systemctl list-unit-files --type=service

# 禁用服务
sudo systemctl disable avahi-daemon
sudo systemctl stop avahi-daemon

# 常见可禁用服务：
# - cups（打印服务）
# - avahi-daemon（网络发现）
# - bluetooth（蓝牙）
```

## 15.7 日志审计

### 日志管理

```bash
# 查看系统日志
sudo journalctl -xe
sudo journalctl -b  # 本次启动
sudo journalctl -f  # 实时查看

# 查看认证日志
sudo cat /var/log/auth.log
sudo grep -i "failed" /var/log/auth.log

# 查看登录记录
last          # 成功登录
lastb         # 失败登录
who           # 当前登录用户
w             # 详细信息

# 查看sudo使用
sudo cat /var/log/sudo.log
```

### auditd - 审计系统

```bash
# 安装
sudo apt install auditd audispd-plugins

# 查看规则
sudo auditctl -l

# 监控文件访问
sudo auditctl -w /etc/passwd -p wa -k passwd_changes

# 监控命令执行
sudo auditctl -a exit,always -F arch=b64 -S execve -k exec_audit

# 搜索审计日志
sudo ausearch -k passwd_changes
sudo ausearch -ts today -k exec_audit

# 审计报告
sudo aureport --summary
sudo aureport --auth
```

## 15.8 数据加密

### 加密分区（LUKS）

```bash
# 安装cryptsetup
sudo apt install cryptsetup

# 创建加密分区
sudo cryptsetup luksFormat /dev/sdb1

# 打开加密分区
sudo cryptsetup luksOpen /dev/sdb1 encrypted_disk

# 格式化
sudo mkfs.ext4 /dev/mapper/encrypted_disk

# 挂载
sudo mount /dev/mapper/encrypted_disk /mnt/encrypted

# 卸载和关闭
sudo umount /mnt/encrypted
sudo cryptsetup luksClose encrypted_disk
```

### 文件加密

```bash
# GPG加密
gpg -c file.txt  # 加密
gpg file.txt.gpg  # 解密

# OpenSSL加密
openssl enc -aes-256-cbc -salt -in file.txt -out file.txt.enc
openssl enc -d -aes-256-cbc -in file.txt.enc -out file.txt

# 加密目录（ecryptfs）
sudo apt install ecryptfs-utils
sudo mount -t ecryptfs /source /target
```

## 15.9 备份和恢复

### 备份策略

```bash
# rsync备份
rsync -avz --delete /source/ /backup/

# 远程备份
rsync -avz -e ssh /source/ user@server:/backup/

# 增量备份脚本
#!/bin/bash
DATE=$(date +%Y%m%d)
rsync -av --link-dest=/backup/latest /source/ /backup/$DATE/
ln -snf /backup/$DATE /backup/latest

# tar备份
tar -czf backup-$DATE.tar.gz /source

# 数据库备份
mysqldump -u root -p database > database-$DATE.sql
pg_dump database > database-$DATE.sql
```

## 15.10 安全检查清单

### 快速安全检查

```bash
#!/bin/bash
# security_check.sh

echo "=== 安全检查 ==="
echo ""

# 1. 检查root登录
echo "检查root远程登录..."
grep "^PermitRootLogin" /etc/ssh/sshd_config || echo "未配置"

# 2. 检查防火墙
echo "检查防火墙状态..."
sudo ufw status

# 3. 检查SUID文件
echo "检查SUID文件..."
find / -perm /4000 -ls 2>/dev/null

# 4. 检查空密码账户
echo "检查空密码账户..."
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# 5. 检查登录失败
echo "最近登录失败..."
sudo lastb | head -10

# 6. 检查监听端口
echo "检查监听端口..."
sudo netstat -tuln | grep LISTEN

# 7. 检查系统更新
echo "检查系统更新..."
sudo apt update && apt list --upgradable

echo ""
echo "检查完成！"
```

## 15.11 应急响应

### 被入侵后的处理

```bash
# 1. 断开网络
sudo ip link set eth0 down

# 2. 备份日志
sudo cp -r /var/log /backup/logs-$(date +%Y%m%d)

# 3. 查找可疑进程
ps aux --sort=-%cpu
netstat -antup

# 4. 查找可疑文件
find /tmp -type f -mtime -1
find /var/tmp -type f -mtime -1

# 5. 检查定时任务
crontab -l
sudo cat /etc/crontab
sudo ls -la /etc/cron.*

# 6. 检查启动项
systemctl list-unit-files | grep enabled

# 7. 查找后门
netstat -antup | grep LISTEN
lsof -i

# 8. 密码重置
sudo passwd username
```

## 15.12 实践练习

### 练习1：配置SSH安全

```bash
# 1. 生成SSH密钥
ssh-keygen -t ed25519

# 2. 配置SSH（禁用密码登录）
sudo nano /etc/ssh/sshd_config
# PasswordAuthentication no

# 3. 配置防火墙
sudo ufw allow 22/tcp
sudo ufw enable
```

### 练习2：配置fail2ban

```bash
# 1. 安装
sudo apt install fail2ban

# 2. 配置
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# 3. 启动
sudo systemctl start fail2ban

# 4. 查看状态
sudo fail2ban-client status
```

### 练习3：系统加固

```bash
# 1. 设置密码策略
sudo nano /etc/login.defs

# 2. 禁用root登录
sudo nano /etc/ssh/sshd_config

# 3. 内核参数加固
sudo nano /etc/sysctl.conf

# 4. 应用更改
sudo sysctl -p
sudo systemctl restart sshd
```

## 下一步

完成本章后，你应该：
- ✅ 掌握Linux安全基本原则
- ✅ 会配置安全的SSH访问
- ✅ 了解防火墙和入侵检测
- ✅ 能够进行系统加固
- ✅ 掌握安全审计方法

**准备好了吗？让我们进入[第16章：容器技术](../16-容器技术/README.md)！**

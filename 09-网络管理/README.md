# 第9章：网络配置与管理

## 9.1 网络基础概念

### IP地址
- **IPv4**: 192.168.1.100
- **IPv6**: 2001:0db8:85a3::8a2e:0370:7334
- **子网掩码**: 255.255.255.0 (/24)
- **网关**: 路由器地址，通常是 192.168.1.1
- **DNS**: 域名解析服务器

### 网络接口
- **lo**: 本地回环 (127.0.0.1)
- **eth0/enp3s0**: 有线网卡
- **wlan0**: 无线网卡
- **docker0**: Docker网桥

## 9.2 查看网络信息

### 使用ip命令（推荐）

```bash
# 查看所有网络接口
ip addr show
ip a

# 查看特定接口
ip addr show eth0

# 查看链路状态
ip link show

# 查看路由表
ip route show
ip r

# 查看ARP表
ip neigh show
```

### 使用ifconfig（旧）

```bash
# 查看所有接口
ifconfig

# 查看特定接口
ifconfig eth0

# 启用/禁用接口
sudo ifconfig eth0 up
sudo ifconfig eth0 down
```

### 其他网络信息命令

```bash
# 查看主机名
hostname
hostname -I    # 显示所有IP

# 查看DNS配置
cat /etc/resolv.conf

# 查看网络统计
netstat -i
ss -s

# 查看路由追踪
traceroute google.com
tracepath google.com
mtr google.com        # 持续追踪
```

## 9.3 配置网络

### Ubuntu/Debian (netplan)

```bash
# 配置文件位置
/etc/netplan/*.yaml

# 示例配置
sudo nano /etc/netplan/01-network-config.yaml

# 静态IP
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# DHCP
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true

# 应用配置
sudo netplan apply

# 测试配置
sudo netplan try
```

### RHEL/CentOS (NetworkManager)

```bash
# 使用nmcli
nmcli device status
nmcli connection show

# 配置静态IP
sudo nmcli con mod eth0 ipv4.addresses 192.168.1.100/24
sudo nmcli con mod eth0 ipv4.gateway 192.168.1.1
sudo nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
sudo nmcli con mod eth0 ipv4.method manual

# 重启连接
sudo nmcli con down eth0
sudo nmcli con up eth0
```

### 临时配置（重启后失效）

```bash
# 添加IP地址
sudo ip addr add 192.168.1.100/24 dev eth0

# 删除IP地址
sudo ip addr del 192.168.1.100/24 dev eth0

# 启用/禁用接口
sudo ip link set eth0 up
sudo ip link set eth0 down

# 添加路由
sudo ip route add default via 192.168.1.1

# 删除路由
sudo ip route del default via 192.168.1.1
```

## 9.4 DNS配置

```bash
# DNS配置文件
sudo nano /etc/resolv.conf

# 内容
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1

# 测试DNS解析
nslookup google.com
dig google.com
host google.com

# 清除DNS缓存
sudo systemd-resolve --flush-caches
```

### hosts文件

```bash
# 编辑hosts
sudo nano /etc/hosts

# 格式
127.0.0.1   localhost
192.168.1.10  server1.local server1
192.168.1.20  server2.local server2

# 优先级：/etc/hosts 优先于 DNS
```

## 9.5 端口和连接

### 查看端口

```bash
# 查看监听端口
sudo netstat -tuln
sudo ss -tuln

# 查看所有连接
sudo netstat -tuan
sudo ss -tuan

# 查看特定端口
sudo netstat -tuln | grep :80
sudo ss -tuln | grep :80

# 查看进程端口
sudo netstat -tulnp
sudo ss -tulnp
sudo lsof -i :80
```

### 端口说明
- `-t`: TCP
- `-u`: UDP
- `-l`: 监听状态
- `-n`: 数字格式
- `-p`: 显示进程

### 测试端口连通性

```bash
# telnet
telnet google.com 80

# nc (netcat)
nc -zv google.com 80
nc -zv 192.168.1.1 22

# 扫描端口范围
nc -zv 192.168.1.1 20-30
```

## 9.6 防火墙

### UFW (Ubuntu)

```bash
# 安装
sudo apt install ufw

# 基本操作
sudo ufw status
sudo ufw enable
sudo ufw disable

# 默认策略
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 允许端口
sudo ufw allow 22/tcp        # SSH
sudo ufw allow 80/tcp         # HTTP
sudo ufw allow 443/tcp        # HTTPS
sudo ufw allow 3306/tcp       # MySQL

# 允许特定IP
sudo ufw allow from 192.168.1.100

# 允许IP访问特定端口
sudo ufw allow from 192.168.1.100 to any port 22

# 拒绝端口
sudo ufw deny 23/tcp

# 删除规则
sudo ufw delete allow 80/tcp

# 查看规则编号
sudo ufw status numbered

# 删除规则（按编号）
sudo ufw delete 2

# 重置防火墙
sudo ufw reset
```

### firewalld (RHEL/CentOS)

```bash
# 状态
sudo firewall-cmd --state

# 列出规则
sudo firewall-cmd --list-all

# 允许服务
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent

# 允许端口
sudo firewall-cmd --add-port=8080/tcp --permanent

# 重载规则
sudo firewall-cmd --reload

# 查看可用服务
sudo firewall-cmd --get-services
```

### iptables（高级）

```bash
# 查看规则
sudo iptables -L -n -v

# 允许SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 允许HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 允许特定IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# 拒绝所有其他
sudo iptables -A INPUT -j DROP

# 保存规则
sudo iptables-save > /etc/iptables/rules.v4

# 恢复规则
sudo iptables-restore < /etc/iptables/rules.v4
```

## 9.7 SSH

### 基本使用

```bash
# 连接服务器
ssh user@hostname
ssh user@192.168.1.100
ssh -p 2222 user@host        # 指定端口

# 执行远程命令
ssh user@host 'ls -l'
ssh user@host 'df -h'

# 传输文件
scp file.txt user@host:/path/
scp user@host:/path/file.txt ./
scp -r directory/ user@host:/path/

# 同步目录
rsync -avz source/ user@host:dest/
rsync -avz --delete source/ user@host:dest/
```

### SSH密钥认证

```bash
# 生成密钥对
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 复制公钥到服务器
ssh-copy-id user@host

# 手动复制
cat ~/.ssh/id_ed25519.pub | ssh user@host "cat >> ~/.ssh/authorized_keys"

# 配置SSH客户端
nano ~/.ssh/config

# 示例配置
Host myserver
    HostName 192.168.1.100
    User myuser
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# 使用：ssh myserver
```

### SSH服务器配置

```bash
# 编辑配置
sudo nano /etc/ssh/sshd_config

# 推荐设置
Port 22                        # 可改为非标准端口
PermitRootLogin no             # 禁止root登录
PasswordAuthentication yes     # 密码认证（可禁用）
PubkeyAuthentication yes       # 密钥认证
AllowUsers user1 user2         # 只允许特定用户
MaxAuthTries 3                 # 最大尝试次数

# 重启SSH服务
sudo systemctl restart sshd
sudo systemctl reload sshd
```

## 9.8 网络工具

### ping - 测试连通性

```bash
ping google.com
ping -c 4 google.com           # 发送4个包
ping -i 2 google.com           # 间隔2秒
ping -s 1000 google.com        # 包大小1000字节
```

### wget - 下载工具

```bash
wget https://example.com/file.zip
wget -O custom.zip https://...    # 自定义文件名
wget -c https://...                # 断点续传
wget -r https://example.com        # 递归下载
wget -b https://...                # 后台下载
```

### curl - HTTP客户端

```bash
curl https://api.example.com
curl -o file.html https://...           # 保存到文件
curl -O https://.../file.zip            # 使用远程文件名
curl -I https://example.com             # 只获取头
curl -L https://...                     # 跟随重定向
curl -X POST https://api.example.com    # POST请求
curl -d "key=value" https://...         # 发送数据
curl -H "Authorization: Bearer token"   # 自定义头
```

### tcpdump - 抓包

```bash
# 抓取所有包
sudo tcpdump

# 抓取特定接口
sudo tcpdump -i eth0

# 抓取特定主机
sudo tcpdump host 192.168.1.100

# 抓取特定端口
sudo tcpdump port 80

# 保存到文件
sudo tcpdump -w capture.pcap

# 读取文件
sudo tcpdump -r capture.pcap

# 组合条件
sudo tcpdump -i eth0 'tcp port 80 and host 192.168.1.100'
```

### nmap - 端口扫描

```bash
# 扫描主机
nmap 192.168.1.100

# 扫描端口范围
nmap -p 1-1000 192.168.1.100

# 扫描子网
nmap 192.168.1.0/24

# 服务版本探测
nmap -sV 192.168.1.100

# OS检测
sudo nmap -O 192.168.1.100
```

## 9.9 网络故障排查

### 步骤1：检查物理连接

```bash
# 查看网卡状态
ip link show
ethtool eth0
```

### 步骤2：检查IP配置

```bash
ip addr show
ip route show
cat /etc/resolv.conf
```

### 步骤3：测试连通性

```bash
# 本地回环
ping 127.0.0.1

# 本机IP
ping $(hostname -I)

# 网关
ping 192.168.1.1

# 外网IP
ping 8.8.8.8

# 域名
ping google.com
```

### 步骤4：检查路由

```bash
ip route show
traceroute google.com
mtr google.com
```

### 步骤5：检查DNS

```bash
nslookup google.com
dig google.com
cat /etc/resolv.conf
```

### 步骤6：检查防火墙

```bash
sudo ufw status
sudo iptables -L -n
```

### 步骤7：检查端口

```bash
sudo netstat -tuln | grep :80
sudo ss -tuln | grep :80
```

## 9.10 实战练习

### 练习1：配置静态IP

```bash
# 1. 备份配置
sudo cp /etc/netplan/01-network-config.yaml /etc/netplan/01-network-config.yaml.bak

# 2. 编辑配置
sudo nano /etc/netplan/01-network-config.yaml

# 3. 应用
sudo netplan apply

# 4. 验证
ip addr show
ping 8.8.8.8
```

### 练习2：配置防火墙

```bash
# 1. 启用UFW
sudo ufw enable

# 2. 允许SSH
sudo ufw allow 22/tcp

# 3. 允许Web服务
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 4. 检查规则
sudo ufw status verbose
```

### 练习3：SSH密钥认证

```bash
# 1. 生成密钥
ssh-keygen -t ed25519

# 2. 复制到服务器
ssh-copy-id user@server

# 3. 测试
ssh user@server

# 4. 禁用密码登录
sudo nano /etc/ssh/sshd_config
# PasswordAuthentication no
sudo systemctl reload sshd
```

## 学习资源

- `man ip`
- `man ssh`
- `man ufw`
- [Linux Network Administrators Guide](https://tldp.org/LDP/nag2/)
- [Netplan Documentation](https://netplan.io/)

**下一章：[第10章：存储管理](../10-存储管理/README.md)**

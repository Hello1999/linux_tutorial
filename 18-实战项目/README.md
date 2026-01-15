# 第18章：实战项目

通过实战项目巩固所学知识！

## 项目1：搭建个人博客

### 目标
使用Nginx + WordPress搭建个人博客

### 步骤

**1. 安装LEMP栈（Linux + Nginx + MySQL + PHP）**

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装Nginx
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx

# 安装MySQL
sudo apt install mysql-server -y
sudo mysql_secure_installation

# 安装PHP
sudo apt install php-fpm php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc -y
```

**2. 配置MySQL数据库**

```bash
sudo mysql

CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'strong_password';
GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**3. 下载WordPress**

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
sudo chown -R www-data:www-data /var/www/html
```

**4. 配置Nginx**

```bash
sudo nano /etc/nginx/sites-available/wordpress

# 添加配置
server {
    listen 80;
    server_name your_domain.com;
    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php-fpm.sock;
    }
}

# 启用站点
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**5. 完成WordPress安装**
访问 `http://your_server_ip` 完成Web界面安装

### 扩展
- 配置SSL证书（Let's Encrypt）
- 设置自动备份
- 性能优化（缓存、CDN）

---

## 项目2：自动化备份系统

### 目标
创建一个自动备份系统，定期备份重要文件到本地和远程服务器

### 脚本：backup_system.sh

```bash
#!/bin/bash

# 配置文件
CONFIG_FILE="/etc/backup_system.conf"

# 读取配置
if [ -f "$CONFIG_FILE" ]; then
    source "$CONFIG_FILE"
else
    echo "配置文件不存在，使用默认配置"
    SOURCE_DIRS="/home /etc /var/www"
    LOCAL_BACKUP_DIR="/backup/local"
    REMOTE_BACKUP_DIR="/backup/remote"
    REMOTE_SERVER="user@backup.server.com"
    RETENTION_DAYS=30
    LOG_FILE="/var/log/backup_system.log"
fi

# 日志函数
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# 创建备份目录
mkdir -p "$LOCAL_BACKUP_DIR"

# 生成备份文件名
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
HOSTNAME=$(hostname)
BACKUP_FILE="backup_${HOSTNAME}_${TIMESTAMP}.tar.gz"

log "开始备份..."

# 本地备份
log "创建本地备份: $BACKUP_FILE"
if tar -czf "${LOCAL_BACKUP_DIR}/${BACKUP_FILE}" $SOURCE_DIRS 2>/dev/null; then
    log "本地备份成功"
    BACKUP_SIZE=$(du -h "${LOCAL_BACKUP_DIR}/${BACKUP_FILE}" | cut -f1)
    log "备份文件大小: $BACKUP_SIZE"
else
    log "本地备份失败"
    exit 1
fi

# 远程备份
if [ -n "$REMOTE_SERVER" ]; then
    log "开始远程备份..."
    if rsync -avz "${LOCAL_BACKUP_DIR}/${BACKUP_FILE}" "${REMOTE_SERVER}:${REMOTE_BACKUP_DIR}/"; then
        log "远程备份成功"
    else
        log "远程备份失败"
    fi
fi

# 清理旧备份
log "清理超过 ${RETENTION_DAYS} 天的旧备份..."
find "$LOCAL_BACKUP_DIR" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

# 生成备份报告
REPORT_FILE="${LOCAL_BACKUP_DIR}/backup_report_${TIMESTAMP}.txt"
cat > "$REPORT_FILE" << EOF
备份报告
========
备份时间: $(date)
主机名: $HOSTNAME
备份文件: $BACKUP_FILE
备份大小: $BACKUP_SIZE
源目录: $SOURCE_DIRS
备份状态: 成功

当前备份列表:
$(ls -lh "$LOCAL_BACKUP_DIR" | grep backup_)
EOF

log "备份完成！报告已生成: $REPORT_FILE"

# 发送邮件通知（可选）
if command -v mail &> /dev/null; then
    cat "$REPORT_FILE" | mail -s "备份报告 - $HOSTNAME" admin@example.com
fi
```

### 配置文件：/etc/backup_system.conf

```bash
SOURCE_DIRS="/home /etc /var/www"
LOCAL_BACKUP_DIR="/backup/local"
REMOTE_BACKUP_DIR="/backup/remote"
REMOTE_SERVER="user@backup-server.com"
RETENTION_DAYS=30
LOG_FILE="/var/log/backup_system.log"
```

### 设置定时任务

```bash
# 编辑crontab
sudo crontab -e

# 每天凌晨2点执行备份
0 2 * * * /usr/local/bin/backup_system.sh

# 每周日凌晨3点完整备份
0 3 * * 0 /usr/local/bin/backup_system.sh full
```

---

## 项目3：服务器监控系统

### 目标
创建一个实时监控系统资源的工具

### monitor.sh

```bash
#!/bin/bash

# 监控配置
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=85
LOG_FILE="/var/log/system_monitor.log"
ALERT_EMAIL="admin@example.com"

# 获取系统信息
get_cpu_usage() {
    top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1
}

get_memory_usage() {
    free | grep Mem | awk '{printf "%.0f", $3/$2 * 100}'
}

get_disk_usage() {
    df -h / | tail -1 | awk '{print $5}' | sed 's/%//'
}

get_load_average() {
    uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//'
}

# 发送告警
send_alert() {
    local subject="$1"
    local message="$2"

    echo "$message" | mail -s "$subject" "$ALERT_EMAIL"
    echo "[ALERT] $(date): $subject - $message" >> "$LOG_FILE"
}

# 主监控循环
while true; do
    CPU=$(get_cpu_usage)
    MEMORY=$(get_memory_usage)
    DISK=$(get_disk_usage)
    LOAD=$(get_load_average)

    # 记录日志
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] CPU: ${CPU}% | MEM: ${MEMORY}% | DISK: ${DISK}% | LOAD: ${LOAD}" >> "$LOG_FILE"

    # 检查CPU
    if (( $(echo "$CPU > $CPU_THRESHOLD" | bc -l) )); then
        send_alert "CPU告警" "CPU使用率: ${CPU}% 超过阈值 ${CPU_THRESHOLD}%"
    fi

    # 检查内存
    if [ $MEMORY -gt $MEMORY_THRESHOLD ]; then
        send_alert "内存告警" "内存使用率: ${MEMORY}% 超过阈值 ${MEMORY_THRESHOLD}%"
    fi

    # 检查磁盘
    if [ $DISK -gt $DISK_THRESHOLD ]; then
        send_alert "磁盘告警" "磁盘使用率: ${DISK}% 超过阈值 ${DISK_THRESHOLD}%"
    fi

    sleep 60  # 每60秒检查一次
done
```

### 可视化监控（使用Grafana + Prometheus）

```bash
# 安装Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.40.0/prometheus-2.40.0.linux-amd64.tar.gz
tar xzf prometheus-2.40.0.linux-amd64.tar.gz
cd prometheus-2.40.0.linux-amd64
./prometheus --config.file=prometheus.yml

# 安装Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xzf node_exporter-1.5.0.linux-amd64.tar.gz
cd node_exporter-1.5.0.linux-amd64
./node_exporter

# 安装Grafana（Debian/Ubuntu）
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt-get update
sudo apt-get install grafana
sudo systemctl start grafana-server
```

---

## 项目4：日志分析工具

### 目标
分析Web服务器日志，生成访问统计报告

### log_analyzer.sh

```bash
#!/bin/bash

# 配置
LOG_FILE="/var/log/nginx/access.log"
REPORT_DIR="/var/www/html/reports"
REPORT_FILE="${REPORT_DIR}/log_report_$(date +%Y%m%d).html"

# 创建报告目录
mkdir -p "$REPORT_DIR"

# 生成HTML报告
cat > "$REPORT_FILE" << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>日志分析报告</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; margin: 20px 0; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
        tr:nth-child(even) { background-color: #f2f2f2; }
        h1 { color: #333; }
        .stat { background-color: #e7f3fe; padding: 10px; margin: 10px 0; border-left: 4px solid #2196F3; }
    </style>
</head>
<body>
    <h1>Web服务器日志分析报告</h1>
    <p>生成时间: $(date)</p>
    <p>日志文件: $LOG_FILE</p>
EOF

# 基本统计
echo '<div class="stat">' >> "$REPORT_FILE"
echo "<h2>基本统计</h2>" >> "$REPORT_FILE"
echo "<p>总访问量: $(wc -l < "$LOG_FILE")</p>" >> "$REPORT_FILE"
echo "<p>独立IP数: $(awk '{print $1}' "$LOG_FILE" | sort -u | wc -l)</p>" >> "$REPORT_FILE"
echo '</div>' >> "$REPORT_FILE"

# Top 10 访问IP
echo "<h2>Top 10 访问IP</h2>" >> "$REPORT_FILE"
echo "<table>" >> "$REPORT_FILE"
echo "<tr><th>IP地址</th><th>访问次数</th></tr>" >> "$REPORT_FILE"
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10 | while read count ip; do
    echo "<tr><td>$ip</td><td>$count</td></tr>" >> "$REPORT_FILE"
done
echo "</table>" >> "$REPORT_FILE"

# Top 10 访问页面
echo "<h2>Top 10 访问页面</h2>" >> "$REPORT_FILE"
echo "<table>" >> "$REPORT_FILE"
echo "<tr><th>页面</th><th>访问次数</th></tr>" >> "$REPORT_FILE"
awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10 | while read count page; do
    echo "<tr><td>$page</td><td>$count</td></tr>" >> "$REPORT_FILE"
done
echo "</table>" >> "$REPORT_FILE"

# HTTP状态码统计
echo "<h2>HTTP状态码统计</h2>" >> "$REPORT_FILE"
echo "<table>" >> "$REPORT_FILE"
echo "<tr><th>状态码</th><th>次数</th></tr>" >> "$REPORT_FILE"
awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn | while read count code; do
    echo "<tr><td>$code</td><td>$count</td></tr>" >> "$REPORT_FILE"
done
echo "</table>" >> "$REPORT_FILE"

# 每小时访问量
echo "<h2>每小时访问量</h2>" >> "$REPORT_FILE"
echo "<table>" >> "$REPORT_FILE"
echo "<tr><th>小时</th><th>访问量</th></tr>" >> "$REPORT_FILE"
awk '{print $4}' "$LOG_FILE" | cut -d: -f2 | sort | uniq -c | while read count hour; do
    echo "<tr><td>${hour}:00</td><td>$count</td></tr>" >> "$REPORT_FILE"
done
echo "</table>" >> "$REPORT_FILE"

# 结束HTML
cat >> "$REPORT_FILE" << 'EOF'
</body>
</html>
EOF

echo "报告已生成: $REPORT_FILE"
```

---

## 项目5：容器化应用部署

### 目标
使用Docker部署一个Python Web应用

### 1. 创建应用

**app.py**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Docker!"

@app.route('/health')
def health():
    return {"status": "healthy"}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**requirements.txt**
```
Flask==2.0.1
```

### 2. 创建Dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

### 3. 构建和运行

```bash
# 构建镜像
docker build -t myapp:v1 .

# 运行容器
docker run -d -p 5000:5000 --name myapp myapp:v1

# 查看日志
docker logs myapp

# 测试
curl http://localhost:5000
```

### 4. Docker Compose多容器部署

**docker-compose.yml**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
    depends_on:
      - db
    restart: always

  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - web
    restart: always

volumes:
  postgres_data:
```

### 5. 部署

```bash
# 启动所有服务
docker-compose up -d

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down
```

---

## 项目6：配置管理和自动化

### 目标
使用Ansible自动化配置多台服务器

### 1. 安装Ansible

```bash
sudo apt install ansible -y
```

### 2. Inventory文件

**hosts.ini**
```ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[databases]
db1 ansible_host=192.168.1.20

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 3. Playbook

**setup_webserver.yml**
```yaml
---
- name: 配置Web服务器
  hosts: webservers
  become: yes

  tasks:
    - name: 更新系统
      apt:
        update_cache: yes
        upgrade: dist

    - name: 安装Nginx
      apt:
        name: nginx
        state: present

    - name: 启动Nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: 复制网站文件
      copy:
        src: ./website/
        dest: /var/www/html/
        owner: www-data
        group: www-data

    - name: 配置防火墙
      ufw:
        rule: allow
        port: '80'
        proto: tcp
```

### 4. 运行

```bash
# 测试连接
ansible all -i hosts.ini -m ping

# 运行playbook
ansible-playbook -i hosts.ini setup_webserver.yml

# 只在特定主机运行
ansible-playbook -i hosts.ini setup_webserver.yml --limit web1
```

---

## 更多项目建议

1. **Git服务器** - 搭建私有Git仓库（GitLab/Gitea）
2. **VPN服务器** - 配置WireGuard VPN
3. **文件同步服务** - Nextcloud私有云
4. **家庭媒体中心** - Plex/Jellyfin
5. **持续集成** - Jenkins CI/CD流水线
6. **数据库集群** - MySQL主从复制/Redis集群
7. **负载均衡器** - Nginx/HAProxy负载均衡
8. **日志收集** - ELK Stack (Elasticsearch, Logstash, Kibana)
9. **容器编排** - Kubernetes集群
10. **安全扫描** - 漏洞扫描和安全加固

## 学习建议

1. **从小项目开始** - 先完成简单项目再挑战复杂项目
2. **记录过程** - 写技术博客记录遇到的问题和解决方案
3. **版本控制** - 所有配置和脚本都用Git管理
4. **自动化优先** - 能自动化的就不要手动操作
5. **安全第一** - 始终考虑安全性
6. **监控和日志** - 为项目添加监控和日志
7. **文档化** - 编写清晰的文档
8. **备份备份备份** - 定期备份重要数据

## 下一步

选择一个项目开始实践吧！遇到问题：
1. 查看日志文件
2. Google/Stack Overflow搜索错误信息
3. 阅读官方文档
4. 在Linux社区寻求帮助

**记住：实践是最好的老师！**

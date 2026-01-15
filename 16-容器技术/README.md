# 第16章：容器技术

## 16.1 容器概述

### 什么是容器？

容器是一种轻量级的虚拟化技术，将应用程序及其依赖打包在一起，实现：
- 环境隔离
- 快速部署
- 资源高效利用
- 可移植性

### 容器 vs 虚拟机

```
虚拟机架构：
┌─────────┐ ┌─────────┐
│  App A  │ │  App B  │
├─────────┤ ├─────────┤
│Guest OS │ │Guest OS │
├─────────┴─┴─────────┤
│     Hypervisor      │
├─────────────────────┤
│      Host OS        │
└─────────────────────┘

容器架构：
┌─────────┐ ┌─────────┐
│  App A  │ │  App B  │
│  +Libs  │ │  +Libs  │
├─────────┴─┴─────────┤
│  Container Runtime  │
├─────────────────────┤
│      Host OS        │
└─────────────────────┘
```

| 特性 | 容器 | 虚拟机 |
|-----|------|--------|
| 启动时间 | 秒级 | 分钟级 |
| 性能 | 接近原生 | 有开销 |
| 磁盘占用 | MB级 | GB级 |
| 隔离性 | 进程级 | 硬件级 |
| 可移植性 | 高 | 中 |

## 16.2 Docker基础

### 安装Docker

```bash
# Ubuntu/Debian
# 1. 更新包索引
sudo apt update

# 2. 安装依赖
sudo apt install apt-transport-https ca-certificates curl software-properties-common

# 3. 添加Docker GPG密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 4. 添加Docker仓库
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. 安装Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 6. 添加用户到docker组（避免每次sudo）
sudo usermod -aG docker $USER
newgrp docker

# 7. 验证安装
docker --version
docker run hello-world
```

### Docker核心概念

**镜像（Image）**：
- 只读模板，包含运行应用所需的一切
- 类似于虚拟机的快照

**容器（Container）**：
- 镜像的运行实例
- 可以启动、停止、删除

**仓库（Registry）**：
- 存储镜像的地方
- Docker Hub是最大的公共仓库

### 基本命令

```bash
# 查看Docker版本
docker --version
docker version
docker info

# 镜像操作
docker images              # 列出镜像
docker pull nginx          # 拉取镜像
docker rmi nginx           # 删除镜像
docker build -t myapp .    # 构建镜像
docker tag myapp myrepo/myapp:v1  # 标记镜像
docker push myrepo/myapp:v1       # 推送镜像

# 容器操作
docker ps                  # 列出运行中的容器
docker ps -a               # 列出所有容器
docker run nginx           # 运行容器
docker start container_id  # 启动容器
docker stop container_id   # 停止容器
docker restart container_id  # 重启容器
docker rm container_id     # 删除容器
docker logs container_id   # 查看日志
docker exec -it container_id bash  # 进入容器
```

### 运行容器

```bash
# 基本运行
docker run nginx

# 后台运行
docker run -d nginx

# 命名容器
docker run --name mynginx nginx

# 端口映射
docker run -p 8080:80 nginx
# 主机8080端口映射到容器80端口

# 卷挂载
docker run -v /host/path:/container/path nginx
docker run -v myvolume:/data nginx

# 环境变量
docker run -e MYSQL_ROOT_PASSWORD=secret mysql

# 交互式运行
docker run -it ubuntu bash

# 自动删除
docker run --rm ubuntu echo "Hello"

# 资源限制
docker run -m 512m --cpus="1.5" nginx

# 综合示例
docker run -d \
  --name myweb \
  -p 8080:80 \
  -v /host/html:/usr/share/nginx/html:ro \
  -e ENV=production \
  --restart unless-stopped \
  nginx
```

### Dockerfile

```dockerfile
# 基础镜像
FROM ubuntu:22.04

# 维护者信息
LABEL maintainer="your@email.com"

# 设置工作目录
WORKDIR /app

# 复制文件
COPY . /app

# 安装依赖
RUN apt-get update && \
    apt-get install -y python3 python3-pip && \
    pip3 install -r requirements.txt && \
    rm -rf /var/lib/apt/lists/*

# 暴露端口
EXPOSE 8000

# 设置环境变量
ENV PYTHONUNBUFFERED=1

# 运行命令
CMD ["python3", "app.py"]
```

**Dockerfile示例（Node.js应用）**：

```dockerfile
FROM node:18-alpine

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install --production

COPY . .

EXPOSE 3000

USER node

CMD ["node", "server.js"]
```

**构建镜像**：

```bash
# 基本构建
docker build -t myapp:v1 .

# 指定Dockerfile
docker build -f Dockerfile.prod -t myapp:v1 .

# 使用构建参数
docker build --build-arg VERSION=1.0 -t myapp:v1 .

# 不使用缓存
docker build --no-cache -t myapp:v1 .
```

## 16.3 Docker网络

### 网络模式

```bash
# 列出网络
docker network ls

# 创建网络
docker network create mynet

# 桥接网络（默认）
docker run --network bridge nginx

# 主机网络（共享主机网络栈）
docker run --network host nginx

# 无网络
docker run --network none nginx

# 自定义网络
docker network create --driver bridge mynet
docker run --network mynet nginx

# 连接容器到网络
docker network connect mynet container_id

# 断开连接
docker network disconnect mynet container_id

# 检查网络
docker network inspect mynet
```

### 容器互联

```bash
# 方法1：使用自定义网络
docker network create mynet
docker run -d --name db --network mynet mysql
docker run -d --name app --network mynet myapp
# app容器可以通过 "db" 访问数据库

# 方法2：使用link（已弃用）
docker run -d --name db mysql
docker run -d --name app --link db:database myapp
```

## 16.4 Docker存储

### 卷（Volume）

```bash
# 创建卷
docker volume create myvolume

# 列出卷
docker volume ls

# 查看卷详情
docker volume inspect myvolume

# 使用卷
docker run -v myvolume:/data nginx

# 匿名卷
docker run -v /data nginx

# 删除卷
docker volume rm myvolume

# 删除所有未使用的卷
docker volume prune
```

### 绑定挂载（Bind Mount）

```bash
# 挂载主机目录
docker run -v /host/path:/container/path nginx

# 只读挂载
docker run -v /host/path:/container/path:ro nginx

# 挂载文件
docker run -v /host/config.conf:/etc/config.conf nginx
```

### tmpfs（临时文件系统）

```bash
# 挂载tmpfs（存储在内存中）
docker run --tmpfs /tmp:rw,size=100m nginx
```

## 16.5 Docker Compose

### 安装Docker Compose

```bash
# Ubuntu（使用apt）
sudo apt install docker-compose-plugin

# 验证
docker compose version
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend
    depends_on:
      - api

  api:
    build: ./api
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    networks:
      - frontend
      - backend
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  db-data:
```

### Docker Compose命令

```bash
# 启动服务
docker compose up

# 后台启动
docker compose up -d

# 构建并启动
docker compose up --build

# 停止服务
docker compose down

# 停止并删除卷
docker compose down -v

# 查看服务
docker compose ps

# 查看日志
docker compose logs
docker compose logs -f web  # 跟踪特定服务

# 执行命令
docker compose exec web bash

# 扩展服务
docker compose up -d --scale api=3

# 重启服务
docker compose restart
```

## 16.6 实用示例

### 示例1：运行MySQL数据库

```bash
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  -e MYSQL_DATABASE=mydb \
  -v mysql-data:/var/lib/mysql \
  --restart unless-stopped \
  mysql:8.0

# 连接数据库
docker exec -it mysql mysql -uroot -p
```

### 示例2：运行WordPress

```yaml
# docker-compose.yml
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: secret
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: wordpress
    volumes:
      - db-data:/var/lib/mysql

volumes:
  wordpress-data:
  db-data:
```

```bash
docker compose up -d
# 访问 http://localhost:8080
```

### 示例3：开发环境（LAMP栈）

```yaml
version: '3.8'

services:
  web:
    image: php:8.1-apache
    ports:
      - "80:80"
    volumes:
      - ./src:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: myapp
    volumes:
      - db-data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin:latest
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db

volumes:
  db-data:
```

## 16.7 Docker最佳实践

### 镜像优化

```dockerfile
# 1. 使用官方基础镜像
FROM node:18-alpine  # alpine版本更小

# 2. 合并RUN命令减少层数
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    rm -rf /var/lib/apt/lists/*

# 3. 使用.dockerignore
# .dockerignore文件：
node_modules
.git
*.log
.env

# 4. 多阶段构建
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm install --production
CMD ["node", "dist/server.js"]

# 5. 使用非root用户
FROM node:18-alpine
RUN addgroup -g 1001 appuser && \
    adduser -D -u 1001 -G appuser appuser
USER appuser
```

### 安全最佳实践

```bash
# 1. 扫描镜像漏洞
docker scan myimage:latest

# 2. 使用官方镜像
docker pull nginx:latest

# 3. 不在镜像中存储敏感信息
# 使用环境变量或secrets
docker run -e DB_PASSWORD=secret myapp

# 4. 限制容器资源
docker run -m 512m --cpus="1" myapp

# 5. 使用只读文件系统
docker run --read-only myapp

# 6. 删除不必要的工具
RUN apt-get remove -y wget curl && \
    rm -rf /var/lib/apt/lists/*
```

## 16.8 Kubernetes入门

### Kubernetes概念

**Pod**：最小部署单元，包含一个或多个容器
**Deployment**：管理Pod的副本
**Service**：Pod的网络访问接口
**Namespace**：资源隔离

### 安装Minikube（本地测试）

```bash
# 安装Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 启动Minikube
minikube start

# 安装kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl

# 验证
kubectl version
kubectl cluster-info
```

### Kubernetes基本命令

```bash
# 创建部署
kubectl create deployment nginx --image=nginx

# 查看部署
kubectl get deployments
kubectl get pods

# 暴露服务
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看服务
kubectl get services

# 扩展副本
kubectl scale deployment nginx --replicas=3

# 查看日志
kubectl logs pod-name

# 进入Pod
kubectl exec -it pod-name -- bash

# 删除资源
kubectl delete deployment nginx
kubectl delete service nginx
```

### YAML配置文件

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

```bash
# 应用配置
kubectl apply -f deployment.yaml

# 删除配置
kubectl delete -f deployment.yaml
```

## 16.9 容器监控

### Docker Stats

```bash
# 实时监控
docker stats

# 监控特定容器
docker stats container_name

# 不显示stream
docker stats --no-stream
```

### cAdvisor

```bash
# 运行cAdvisor
docker run -d \
  --name=cadvisor \
  -p 8080:8080 \
  -v /:/rootfs:ro \
  -v /var/run:/var/run:ro \
  -v /sys:/sys:ro \
  -v /var/lib/docker/:/var/lib/docker:ro \
  gcr.io/cadvisor/cadvisor:latest

# 访问 http://localhost:8080
```

## 16.10 实践练习

### 练习1：运行第一个容器

```bash
# 1. 运行Nginx
docker run -d -p 8080:80 --name myweb nginx

# 2. 访问
curl http://localhost:8080

# 3. 查看日志
docker logs myweb

# 4. 停止并删除
docker stop myweb
docker rm myweb
```

### 练习2：构建自定义镜像

```bash
# 1. 创建目录和文件
mkdir myapp
cd myapp

# 2. 创建Dockerfile
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EOF

# 3. 创建index.html
echo "<h1>Hello Docker!</h1>" > index.html

# 4. 构建镜像
docker build -t myapp:v1 .

# 5. 运行容器
docker run -d -p 8080:80 myapp:v1
```

### 练习3：使用Docker Compose

```bash
# 1. 创建docker-compose.yml（使用上面的WordPress示例）

# 2. 启动服务
docker compose up -d

# 3. 查看状态
docker compose ps

# 4. 查看日志
docker compose logs

# 5. 停止服务
docker compose down
```

## 下一步

完成本章后，你应该：
- ✅ 理解容器技术原理
- ✅ 掌握Docker基本操作
- ✅ 会编写Dockerfile
- ✅ 能够使用Docker Compose
- ✅ 了解Kubernetes基础

**准备好了吗？让我们进入[第17章：高级Shell](../17-高级Shell/README.md)！**

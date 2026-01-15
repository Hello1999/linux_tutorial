# 第7章：软件包管理

## 7.1 软件包管理概述

### 什么是软件包？

软件包是预编译的软件及其依赖文件的集合，包含：
- 可执行文件
- 配置文件
- 文档
- 依赖关系信息
- 安装/卸载脚本

### 包管理系统的作用

- 自动处理依赖关系
- 统一安装/卸载流程
- 版本管理和更新
- 软件源管理
- 验证软件完整性

### 主流包管理系统

| 发行版家族 | 包格式 | 底层工具 | 高级工具 |
|----------|--------|---------|---------|
| Debian/Ubuntu | .deb | dpkg | apt, apt-get |
| Red Hat/Fedora | .rpm | rpm | dnf, yum |
| Arch Linux | .pkg.tar.xz | pacman | pacman |
| SUSE | .rpm | rpm | zypper |

## 7.2 Debian/Ubuntu包管理 (APT)

### APT基础

APT (Advanced Package Tool) 是Debian系发行版的包管理工具。

```bash
# 更新软件包索引
sudo apt update

# 升级所有软件包
sudo apt upgrade

# 升级系统（可能删除旧包）
sudo apt full-upgrade

# 安装软件包
sudo apt install package_name

# 安装多个包
sudo apt install package1 package2 package3

# 重新安装软件包
sudo apt install --reinstall package_name

# 卸载软件包（保留配置文件）
sudo apt remove package_name

# 完全卸载（包括配置文件）
sudo apt purge package_name

# 删除不需要的依赖
sudo apt autoremove

# 清理下载的包文件
sudo apt clean
sudo apt autoclean  # 只清理过时的包
```

### 搜索和查询

```bash
# 搜索软件包
apt search keyword
apt search "web server"

# 显示包信息
apt show package_name

# 列出所有已安装的包
apt list --installed

# 列出可升级的包
apt list --upgradeable

# 列出所有可用版本
apt list -a package_name

# 查看包的依赖关系
apt depends package_name

# 查看哪些包依赖某个包
apt rdepends package_name
```

### apt vs apt-get

`apt` 是更现代的命令，结合了 `apt-get` 和 `apt-cache` 的功能。

```bash
# 推荐使用apt（更友好的输出）
sudo apt install package

# 传统方式（脚本中常用）
sudo apt-get install package
```

### 高级用法

```bash
# 下载包但不安装
apt download package_name

# 模拟安装（不实际安装）
sudo apt install -s package_name

# 安装特定版本
sudo apt install package_name=version

# 从本地deb文件安装
sudo apt install ./package.deb

# 固定包版本（不自动升级）
sudo apt-mark hold package_name

# 取消固定
sudo apt-mark unhold package_name

# 查看被固定的包
apt-mark showhold
```

### 软件源管理

```bash
# 软件源配置文件
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# 添加PPA（Ubuntu）
sudo add-apt-repository ppa:user/ppa-name
sudo apt update

# 删除PPA
sudo add-apt-repository --remove ppa:user/ppa-name

# 编辑软件源
sudo nano /etc/apt/sources.list
```

**国内镜像源**（以Ubuntu 22.04为例）：

```bash
# 阿里云镜像
deb https://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse

# 清华镜像
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
```

### dpkg低级工具

```bash
# 安装deb包
sudo dpkg -i package.deb

# 卸载包
sudo dpkg -r package_name

# 完全卸载
sudo dpkg -P package_name

# 列出已安装的包
dpkg -l
dpkg -l | grep keyword

# 查看包是否安装
dpkg -l package_name
dpkg -s package_name

# 列出包安装的文件
dpkg -L package_name

# 查看文件属于哪个包
dpkg -S /path/to/file

# 查看deb包内容
dpkg -c package.deb

# 解包（不安装）
dpkg -x package.deb /extract/path

# 修复损坏的依赖
sudo apt --fix-broken install
```

## 7.3 Red Hat/Fedora包管理 (DNF/YUM)

### DNF基础

DNF (Dandified YUM) 是YUM的继任者，用于Fedora和新版RHEL/CentOS。

```bash
# 更新包索引
sudo dnf check-update

# 升级所有包
sudo dnf upgrade

# 安装软件包
sudo dnf install package_name

# 安装多个包
sudo dnf install package1 package2

# 重新安装
sudo dnf reinstall package_name

# 卸载软件包
sudo dnf remove package_name

# 删除不需要的依赖
sudo dnf autoremove

# 清理缓存
sudo dnf clean all
```

### 搜索和查询

```bash
# 搜索软件包
dnf search keyword

# 显示包信息
dnf info package_name

# 列出已安装的包
dnf list installed

# 列出可升级的包
dnf list updates

# 列出所有可用包
dnf list available

# 查看依赖关系
dnf deplist package_name

# 查看哪个包提供某个文件
dnf provides /usr/bin/python3

# 查看包的文件列表
dnf repoquery -l package_name
```

### 软件源管理

```bash
# 列出所有软件源
dnf repolist

# 列出所有软件源（包括禁用的）
dnf repolist all

# 启用软件源
sudo dnf config-manager --enable repo_name

# 禁用软件源
sudo dnf config-manager --disable repo_name

# 添加软件源
sudo dnf config-manager --add-repo https://example.com/repo

# 查看软件源配置
cat /etc/yum.repos.d/*.repo
```

**EPEL仓库**（额外的包）：

```bash
# RHEL/CentOS
sudo dnf install epel-release

# 或手动添加
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

### 包组管理

```bash
# 列出可用的包组
dnf group list

# 安装包组
sudo dnf group install "Development Tools"

# 卸载包组
sudo dnf group remove "Development Tools"

# 查看包组信息
dnf group info "Development Tools"
```

### RPM低级工具

```bash
# 安装rpm包
sudo rpm -ivh package.rpm

# 升级rpm包
sudo rpm -Uvh package.rpm

# 卸载包
sudo rpm -e package_name

# 查询已安装的包
rpm -qa
rpm -qa | grep keyword

# 查看包是否安装
rpm -q package_name

# 查看包信息
rpm -qi package_name

# 列出包安装的文件
rpm -ql package_name

# 查看文件属于哪个包
rpm -qf /path/to/file

# 查看rpm包内容
rpm -qpl package.rpm

# 验证包的完整性
rpm -V package_name
```

### YUM（旧版）

在CentOS 7及更早版本中使用YUM：

```bash
# 基本命令（与dnf类似）
sudo yum install package_name
sudo yum update
sudo yum remove package_name
sudo yum search keyword
yum info package_name
```

## 7.4 Arch Linux包管理 (Pacman)

### Pacman基础

```bash
# 同步包数据库并升级系统
sudo pacman -Syu

# 只同步包数据库
sudo pacman -Sy

# 安装软件包
sudo pacman -S package_name

# 安装多个包
sudo pacman -S package1 package2

# 重新安装
sudo pacman -S --overwrite '*' package_name

# 卸载软件包
sudo pacman -R package_name

# 卸载包及其依赖
sudo pacman -Rs package_name

# 卸载包及其配置文件
sudo pacman -Rn package_name

# 完全卸载
sudo pacman -Rns package_name

# 清理缓存
sudo pacman -Sc      # 清理未安装的包
sudo pacman -Scc     # 清理所有缓存

# 删除孤立包
sudo pacman -Rns $(pacman -Qtdq)
```

### 搜索和查询

```bash
# 搜索软件包
pacman -Ss keyword

# 搜索已安装的包
pacman -Qs keyword

# 显示包信息
pacman -Si package_name      # 仓库中的包
pacman -Qi package_name      # 已安装的包

# 列出已安装的包
pacman -Q

# 列出显式安装的包
pacman -Qe

# 列出依赖安装的包
pacman -Qd

# 查看包的文件列表
pacman -Ql package_name

# 查看文件属于哪个包
pacman -Qo /path/to/file

# 查看孤立包
pacman -Qdt
```

### Pacman选项说明

```bash
-S      # Sync（同步/安装）
-R      # Remove（卸载）
-Q      # Query（查询已安装）
-U      # Upgrade（从文件升级）
-F      # Files（文件数据库）

-y      # 刷新数据库
-u      # 升级
-s      # 搜索
-i      # 信息
-c      # 清理
-n      # 删除配置文件
```

### AUR (Arch User Repository)

AUR是用户贡献的软件包仓库，需要使用AUR助手。

**使用yay（AUR助手）**：

```bash
# 安装yay
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

# 使用yay（命令与pacman类似）
yay -S package_name      # 安装
yay -Syu                 # 更新系统和AUR包
yay -Ss keyword          # 搜索
yay -R package_name      # 卸载
```

## 7.5 通用包管理 - Snap

Snap是跨发行版的通用包管理系统，由Canonical开发。

### Snap基础

```bash
# 安装snapd
sudo apt install snapd          # Ubuntu/Debian
sudo dnf install snapd          # Fedora
sudo pacman -S snapd            # Arch

# 启动snapd服务
sudo systemctl enable --now snapd.socket

# 搜索snap包
snap find keyword

# 安装snap包
sudo snap install package_name

# 安装特定频道的包
sudo snap install package_name --classic
sudo snap install package_name --edge

# 列出已安装的snap
snap list

# 更新所有snap
sudo snap refresh

# 更新特定snap
sudo snap refresh package_name

# 卸载snap
sudo snap remove package_name

# 查看snap信息
snap info package_name

# 回退到上一个版本
sudo snap revert package_name
```

### Snap频道

```bash
stable      # 稳定版（默认）
candidate   # 候选版
beta        # 测试版
edge        # 开发版
```

## 7.6 通用包管理 - Flatpak

Flatpak是另一个跨发行版的包管理系统。

### Flatpak基础

```bash
# 安装flatpak
sudo apt install flatpak        # Ubuntu/Debian
sudo dnf install flatpak        # Fedora
sudo pacman -S flatpak          # Arch

# 添加Flathub仓库
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# 搜索应用
flatpak search keyword

# 安装应用
flatpak install flathub app.name

# 运行应用
flatpak run app.name

# 列出已安装的应用
flatpak list

# 更新所有应用
flatpak update

# 卸载应用
flatpak uninstall app.name

# 删除未使用的运行时
flatpak uninstall --unused

# 查看应用信息
flatpak info app.name
```

## 7.7 从源码编译安装

有时需要从源码编译软件。

### 标准编译流程

```bash
# 1. 安装编译工具
# Ubuntu/Debian
sudo apt install build-essential

# Fedora
sudo dnf groupinstall "Development Tools"

# Arch
sudo pacman -S base-devel

# 2. 下载源码
wget https://example.com/software-1.0.tar.gz
tar -xzf software-1.0.tar.gz
cd software-1.0

# 3. 配置
./configure --prefix=/usr/local

# 常用配置选项
./configure --help              # 查看所有选项
./configure --prefix=/opt/app   # 指定安装目录
./configure --enable-feature    # 启用特性
./configure --disable-feature   # 禁用特性

# 4. 编译
make

# 使用多核编译（加速）
make -j$(nproc)

# 5. 安装
sudo make install

# 6. 卸载（如果支持）
sudo make uninstall
```

### CMake项目

```bash
# 1. 创建构建目录
mkdir build
cd build

# 2. 配置
cmake ..
# 或指定安装路径
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..

# 3. 编译
make -j$(nproc)

# 4. 安装
sudo make install
```

### 安装依赖

```bash
# 查看编译依赖
./configure
# 根据错误信息安装缺失的库

# Ubuntu/Debian - 安装开发库
sudo apt install libssl-dev
sudo apt install build-essential

# Fedora
sudo dnf install openssl-devel
sudo dnf groupinstall "Development Tools"
```

## 7.8 AppImage

AppImage是便携式应用程序格式，无需安装。

```bash
# 1. 下载AppImage文件
wget https://example.com/App.AppImage

# 2. 添加执行权限
chmod +x App.AppImage

# 3. 运行
./App.AppImage

# 集成到系统（可选）
./App.AppImage --appimage-extract
# 手动创建桌面文件
```

## 7.9 软件包管理最佳实践

### 系统维护

```bash
# 定期更新系统
# Ubuntu/Debian
sudo apt update && sudo apt upgrade

# Fedora
sudo dnf upgrade

# Arch
sudo pacman -Syu

# 清理不需要的包
sudo apt autoremove      # Ubuntu/Debian
sudo dnf autoremove      # Fedora
sudo pacman -Rns $(pacman -Qtdq)  # Arch

# 清理包缓存
sudo apt clean           # Ubuntu/Debian
sudo dnf clean all       # Fedora
sudo pacman -Sc          # Arch
```

### 安全建议

```bash
# 只从官方仓库安装
# 验证软件包签名
# 不要随意添加第三方源
# 定期更新系统补丁

# 在Ubuntu上启用自动安全更新
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

### 常见问题

**依赖问题**：

```bash
# Ubuntu/Debian
sudo apt --fix-broken install
sudo dpkg --configure -a

# Fedora
sudo dnf distro-sync
```

**包损坏**：

```bash
# Ubuntu/Debian
sudo apt install --reinstall package_name

# Fedora
sudo dnf reinstall package_name

# Arch
sudo pacman -S --overwrite '*' package_name
```

**锁定文件问题**：

```bash
# Ubuntu/Debian
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/cache/apt/archives/lock
sudo dpkg --configure -a

# Fedora
sudo rm /var/cache/dnf/metadata_lock.pid
```

## 7.10 包管理对比

| 功能 | APT | DNF | Pacman |
|-----|-----|-----|--------|
| 搜索 | apt search | dnf search | pacman -Ss |
| 安装 | apt install | dnf install | pacman -S |
| 卸载 | apt remove | dnf remove | pacman -R |
| 更新索引 | apt update | dnf check-update | pacman -Sy |
| 升级系统 | apt upgrade | dnf upgrade | pacman -Syu |
| 包信息 | apt show | dnf info | pacman -Si |
| 列出文件 | dpkg -L | dnf repoquery -l | pacman -Ql |
| 查找文件 | dpkg -S | dnf provides | pacman -Qo |
| 清理缓存 | apt clean | dnf clean all | pacman -Sc |

## 7.11 实用脚本

### 脚本1：系统更新脚本

```bash
#!/bin/bash
# update_system.sh - 自动更新系统

echo "=== 开始系统更新 ==="

# 检测发行版
if [ -f /etc/debian_version ]; then
    echo "检测到Debian/Ubuntu系统"
    sudo apt update
    sudo apt upgrade -y
    sudo apt autoremove -y
    sudo apt clean

elif [ -f /etc/redhat-release ]; then
    echo "检测到Red Hat/Fedora系统"
    sudo dnf upgrade -y
    sudo dnf autoremove -y
    sudo dnf clean all

elif [ -f /etc/arch-release ]; then
    echo "检测到Arch Linux系统"
    sudo pacman -Syu --noconfirm
    sudo pacman -Sc --noconfirm

else
    echo "未识别的系统"
    exit 1
fi

echo "=== 更新完成 ==="
```

### 脚本2：批量安装软件

```bash
#!/bin/bash
# install_essentials.sh - 安装常用软件

PACKAGES=(
    "vim"
    "git"
    "curl"
    "wget"
    "htop"
    "tree"
    "tmux"
)

echo "将安装以下软件："
printf '%s\n' "${PACKAGES[@]}"

read -p "确认安装？(y/n) " -r
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    exit 1
fi

# 检测包管理器
if command -v apt &> /dev/null; then
    sudo apt update
    sudo apt install -y "${PACKAGES[@]}"
elif command -v dnf &> /dev/null; then
    sudo dnf install -y "${PACKAGES[@]}"
elif command -v pacman &> /dev/null; then
    sudo pacman -S --noconfirm "${PACKAGES[@]}"
fi

echo "安装完成！"
```

### 脚本3：清理系统

```bash
#!/bin/bash
# clean_system.sh - 清理系统

echo "=== 开始系统清理 ==="

if [ -f /etc/debian_version ]; then
    echo "清理APT缓存..."
    sudo apt autoremove -y
    sudo apt autoclean
    sudo apt clean

    echo "清理日志..."
    sudo journalctl --vacuum-time=7d

    echo "清理缩略图缓存..."
    rm -rf ~/.cache/thumbnails/*

elif [ -f /etc/arch-release ]; then
    echo "清理Pacman缓存..."
    sudo pacman -Sc --noconfirm
    sudo pacman -Rns $(pacman -Qtdq) --noconfirm 2>/dev/null

    echo "清理日志..."
    sudo journalctl --vacuum-time=7d
fi

echo "清理完成！释放的空间："
df -h /
```

## 7.12 实践练习

### 练习1：基本操作（Ubuntu）

```bash
# 1. 更新系统
sudo apt update

# 2. 安装一个软件
sudo apt install neofetch

# 3. 运行并查看
neofetch

# 4. 卸载软件
sudo apt remove neofetch
sudo apt autoremove
```

### 练习2：搜索和查询

```bash
# 搜索包含"python"的包
apt search python

# 查看python3的详细信息
apt show python3

# 查看python3安装了哪些文件
dpkg -L python3

# 查看/usr/bin/python3属于哪个包
dpkg -S /usr/bin/python3
```

### 练习3：处理依赖问题

```bash
# 模拟依赖问题（不要真的执行）
# sudo dpkg -i --force-all some_package.deb

# 修复依赖
sudo apt --fix-broken install
```

## 下一步

完成本章后，你应该：
- ✅ 掌握主流发行版的包管理工具
- ✅ 理解软件包依赖关系
- ✅ 会从源码编译安装软件
- ✅ 了解通用包格式（Snap、Flatpak）
- ✅ 能够维护和优化系统软件包

**准备好了吗？让我们进入[第8章：进程和服务管理](../08-进程服务/README.md)！**

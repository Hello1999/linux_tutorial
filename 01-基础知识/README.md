# 第1章：Linux基础知识

## 1.1 什么是Linux？

Linux是一个**免费**、**开源**的类Unix操作系统内核，由芬兰程序员Linus Torvalds于1991年创建。

### 核心特点：
- **开源自由**：源代码完全公开，任何人都可以查看、修改和分发
- **多用户多任务**：可以同时运行多个程序，支持多个用户同时使用
- **稳定可靠**：服务器可以连续运行数年不重启
- **安全性高**：权限管理严格，病毒极少
- **可定制性强**：从嵌入式设备到超级计算机都能运行

## 1.2 Linux简史

### 重要时间节点：

**1969年** - UNIX诞生
贝尔实验室的Ken Thompson和Dennis Ritchie创建了UNIX系统

**1983年** - GNU项目启动
Richard Stallman启动GNU计划，目标是创建一个完全自由的类Unix操作系统

**1991年8月25日** - Linux诞生
Linus Torvalds在comp.os.minix新闻组发布了Linux 0.01版本
```
Hello everybody out there using minix -
I'm doing a (free) operating system (just a hobby, won't be big and
professional like gnu) for 386(486) AT clones.
```

**1992年** - Linux采用GPL许可证
Linux内核开始使用GNU GPL协议，真正成为自由软件

**1993年** - Slackware和Debian发行版诞生
第一批真正意义上的Linux发行版出现

**1994年** - Linux 1.0发布
第一个正式版本，包含176,250行代码

**2004年** - Ubuntu发布
让Linux更易用，推动了Linux桌面系统的发展

**今天** - Linux无处不在
- 超过90%的云服务器运行Linux
- Android系统基于Linux内核
- 世界前500的超级计算机100%运行Linux

## 1.3 Linux vs Windows vs macOS

| 特性 | Linux | Windows | macOS |
|------|-------|---------|-------|
| 开源 | ✅ 是 | ❌ 否 | ❌ 否 |
| 价格 | 免费 | 付费 | 随硬件购买 |
| 自定义 | 极高 | 中 | 低 |
| 软件生态 | 丰富（开源为主） | 最丰富 | 丰富 |
| 游戏支持 | 一般（改善中） | 最好 | 一般 |
| 服务器使用 | 主流 | 较少 | 很少 |
| 安全性 | 高 | 中 | 高 |
| 硬件要求 | 低 | 中高 | 高（专用硬件） |
| 学习曲线 | 陡峭 | 平缓 | 平缓 |

## 1.4 Linux系统架构

Linux系统采用层次化架构：

```
┌─────────────────────────────────────────┐
│        用户应用程序层                    │
│  (浏览器、编辑器、数据库等)              │
├─────────────────────────────────────────┤
│        Shell层 (命令行界面)              │
│  (Bash, Zsh, Fish等)                    │
├─────────────────────────────────────────┤
│        系统库层                          │
│  (GNU C Library, 其他系统库)            │
├─────────────────────────────────────────┤
│        系统调用接口                      │
│  (open, read, write, fork等)            │
├─────────────────────────────────────────┤
│        Linux内核                         │
│  ┌────────────────────────────────┐    │
│  │  进程管理  │  内存管理         │    │
│  ├────────────────────────────────┤    │
│  │  文件系统  │  设备驱动         │    │
│  ├────────────────────────────────┤    │
│  │  网络协议栈 │ 进程间通信       │    │
│  └────────────────────────────────┘    │
├─────────────────────────────────────────┤
│        硬件层                            │
│  (CPU、内存、硬盘、网卡等)              │
└─────────────────────────────────────────┘
```

### 内核功能模块：

1. **进程管理** - 创建、调度、终止进程
2. **内存管理** - 虚拟内存、分页、交换
3. **文件系统** - VFS、ext4、XFS等
4. **设备驱动** - 硬件抽象和控制
5. **网络协议栈** - TCP/IP实现
6. **进程间通信(IPC)** - 管道、信号、共享内存

## 1.5 Linux发行版详解

Linux内核 + GNU工具 + 软件包管理器 + 桌面环境 = **Linux发行版**

### 发行版家族树：

```
Linux
├── Debian系
│   ├── Debian
│   ├── Ubuntu
│   │   ├── Linux Mint
│   │   ├── Pop!_OS
│   │   └── elementary OS
│   └── Kali Linux
│
├── Red Hat系
│   ├── Red Hat Enterprise Linux (RHEL)
│   ├── Fedora
│   ├── CentOS (已终止)
│   ├── Rocky Linux
│   └── AlmaLinux
│
├── Arch系
│   ├── Arch Linux
│   ├── Manjaro
│   └── EndeavourOS
│
├── SUSE系
│   ├── openSUSE
│   └── SUSE Linux Enterprise
│
├── Gentoo系
│   ├── Gentoo
│   └── Funtoo
│
└── 独立发行版
    ├── Slackware
    ├── Void Linux
    └── NixOS
```

### 主流发行版选择指南：

#### 🔰 新手友好型：
**Ubuntu**
- 优点：文档丰富、社区活跃、软件多、易上手
- 缺点：有点臃肿
- 适合：完全新手、想快速上手的用户
- 包管理器：apt

**Linux Mint**
- 优点：比Ubuntu更轻量、界面类似Windows、开箱即用
- 缺点：软件更新较Ubuntu稍慢
- 适合：Windows迁移用户
- 包管理器：apt

**Fedora**
- 优点：新技术快速采用、稳定性好
- 缺点：软件仓库比Ubuntu小
- 适合：想尝试新技术的用户
- 包管理器：dnf

#### 🛠️ 服务器/企业级：
**Debian**
- 优点：极其稳定、历史悠久、包数量最多
- 缺点：软件版本较老
- 适合：服务器、追求稳定的生产环境
- 包管理器：apt

**Rocky Linux / AlmaLinux**
- 优点：RHEL的免费替代品、企业级稳定性
- 缺点：软件较旧
- 适合：企业服务器、RHEL迁移用户
- 包管理器：dnf/yum

#### 🚀 进阶用户：
**Arch Linux**
- 优点：滚动更新、极简主义、高度可定制、文档优秀
- 缺点：需要手动配置一切、学习曲线陡峭
- 适合：想深入理解Linux的用户
- 包管理器：pacman

**Manjaro**
- 优点：Arch的用户友好版、滚动更新、开箱即用
- 缺点：比纯Arch稳定性略低
- 适合：想用Arch但怕麻烦的用户
- 包管理器：pacman

#### 🔒 安全/渗透测试：
**Kali Linux**
- 优点：预装上千款安全工具
- 适合：渗透测试人员、安全研究员
- 包管理器：apt

#### 💡 特殊用途：
**Gentoo**
- 特点：所有软件从源码编译、极致性能优化
- 适合：性能狂热者、学习Linux内部机制

**NixOS**
- 特点：声明式配置、可重现构建
- 适合：DevOps工程师

## 1.6 桌面环境

Linux可以自由选择桌面环境（DE）：

### 主流桌面环境：

**GNOME**
- 外观：现代、简洁
- 资源占用：较高（600MB+）
- 特点：触摸屏友好、扩展丰富
- 使用发行版：Ubuntu、Fedora

**KDE Plasma**
- 外观：华丽、可定制性极强
- 资源占用：中等（500MB）
- 特点：类似Windows、功能最丰富
- 使用发行版：Kubuntu、KDE neon

**XFCE**
- 外观：传统、简单
- 资源占用：低（300MB）
- 特点：轻量、稳定
- 使用发行版：Xubuntu

**Cinnamon**
- 外观：传统、优雅
- 资源占用：中等
- 特点：类似Windows 7
- 使用发行版：Linux Mint

**窗口管理器（WM）**
- i3、bspwm、awesome：平铺式窗口管理
- Openbox：轻量级浮动窗口管理
- 资源占用极低（100MB以下）
- 适合：高级用户、追求效率

## 1.7 开源许可证

了解开源许可证对理解Linux生态很重要：

### GPL (GNU General Public License)
- Linux内核使用GPLv2
- **核心原则**：修改后的代码必须继续开源
- **传染性**：衍生作品必须使用相同许可证

### MIT License
- 最宽松的许可证之一
- 可以闭源商用

### Apache License 2.0
- 类似MIT但包含专利授权
- 很多企业项目使用

### BSD License
- 非常自由，允许闭源
- macOS部分基于BSD

## 1.8 Linux哲学

### Unix/Linux设计哲学：

1. **做一件事并做好** - 每个程序只专注一个功能
2. **一切皆文件** - 硬件设备、进程信息都是文件
3. **小程序组合** - 通过管道连接简单程序完成复杂任务
4. **纯文本配置** - 配置文件都是可读的文本
5. **避免强制交互** - 程序应该能够自动化运行

### KISS原则
**Keep It Simple, Stupid**
简单的设计通常是最好的设计

## 1.9 为什么要学习Linux？

### 职业发展：
- 🌐 **云计算时代**：AWS、Azure、GCP都基于Linux
- 💼 **DevOps必备**：CI/CD、容器化、编排都在Linux上
- 🔧 **后端开发**：绝大多数服务器运行Linux
- 🔐 **网络安全**：渗透测试、安全运维离不开Linux
- 📊 **数据科学**：大数据平台基本都是Linux

### 技能提升：
- 深入理解操作系统原理
- 掌握命令行，提高工作效率
- 培养问题解决能力
- 学会阅读文档和源代码

### 实际应用：
- 搭建个人博客/网站
- 配置家庭服务器（NAS、媒体中心）
- 开发嵌入式系统（树莓派等）
- 让老旧电脑焕发新生

## 1.10 第一步：安装Linux

### 方式一：虚拟机（推荐新手）
1. 下载VirtualBox或VMware
2. 下载Ubuntu ISO镜像
3. 创建虚拟机并安装
4. 优点：安全、可以随意实验

### 方式二：双系统
1. 使用U盘制作启动盘
2. 调整硬盘分区
3. 安装Linux
4. 优点：性能完整、真实体验

### 方式三：WSL2（Windows用户）
1. 启用Windows Subsystem for Linux
2. 从Microsoft Store安装Ubuntu
3. 优点：方便快捷、与Windows无缝集成
4. 缺点：功能有限制

### 方式四：云服务器
1. 购买VPS（如阿里云、腾讯云、DigitalOcean）
2. 直接使用Linux系统
3. 优点：学习服务器管理
4. 缺点：需要花钱、网络延迟

## 1.11 学习资源推荐

### 在线资源：
- **Linux Journey** - 互动式学习
- **OverTheWire** - 游戏化学习Linux
- **Arch Wiki** - 最详细的Linux文档（即使不用Arch也值得看）
- **Linux From Scratch** - 从零构建Linux系统

### 命令行帮助：
```bash
man 命令名     # 查看详细手册
命令名 --help  # 查看快速帮助
tldr 命令名    # 查看实用示例（需安装tldr）
```

### 书籍：
- 入门：《鸟哥的Linux私房菜》
- 进阶：《Linux命令行与Shell脚本编程大全》
- 深入：《深入理解Linux内核》
- 编程：《Unix环境高级编程》(APUE)

## 1.12 实践练习

### 练习1：探索你的系统
安装Linux后，尝试回答这些问题：
1. 你使用的是哪个发行版？哪个版本？
   ```bash
   cat /etc/os-release
   ```

2. 内核版本是多少？
   ```bash
   uname -r
   ```

3. 桌面环境是什么？
   ```bash
   echo $DESKTOP_SESSION
   ```

4. CPU信息如何？
   ```bash
   lscpu
   ```

### 练习2：更新系统
根据你的发行版，执行系统更新：
```bash
# Debian/Ubuntu
sudo apt update && sudo apt upgrade

# Fedora
sudo dnf upgrade

# Arch
sudo pacman -Syu
```

### 练习3：安装第一个软件
尝试安装一个实用工具：
```bash
# 安装neofetch（显示系统信息的工具）
sudo apt install neofetch    # Debian/Ubuntu
sudo dnf install neofetch    # Fedora
sudo pacman -S neofetch      # Arch

# 运行
neofetch
```

## 下一步

完成本章后，你应该：
- ✅ 理解什么是Linux及其历史
- ✅ 了解主流发行版的区别
- ✅ 知道Linux的基本架构
- ✅ 成功安装Linux系统

**准备好了吗？让我们进入[第2章：文件系统结构](../02-文件系统/README.md)！**

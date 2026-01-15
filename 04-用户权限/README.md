# 第4章：用户和权限管理

## 4.1 Linux用户系统概述

Linux是多用户操作系统，支持多个用户同时登录和使用。

### 用户类型

**1. 超级用户（root）**
- UID = 0
- 拥有系统最高权限
- 可以做任何事情（包括破坏系统）

**2. 系统用户**
- UID 1-999（不同发行版可能不同）
- 用于运行系统服务（如www-data, mysql）
- 通常不能登录

**3. 普通用户**
- UID ≥ 1000
- 日常使用的用户账户
- 权限受限

### 用户标识

```bash
id                    # 查看当前用户信息
# uid=1000(zhang) gid=1000(zhang) groups=1000(zhang),27(sudo)

id username           # 查看指定用户
whoami                # 当前用户名
who                   # 登录的用户
```

## 4.2 用户相关文件

### `/etc/passwd` - 用户账户信息
每行一个用户，格式：
```
username:x:UID:GID:comment:home:shell
```

示例：
```bash
root:x:0:0:root:/root:/bin/bash
zhang:x:1000:1000:Zhang San:/home/zhang:/bin/bash
```

字段说明：
1. **username** - 用户名
2. **x** - 密码占位符（实际密码在/etc/shadow）
3. **UID** - 用户ID
4. **GID** - 主组ID
5. **comment** - 注释（通常是全名）
6. **home** - 主目录路径
7. **shell** - 登录Shell

查看：
```bash
cat /etc/passwd
grep "^zhang" /etc/passwd
```

### `/etc/shadow` - 密码文件（加密）
只有root能读取

```bash
zhang:$6$rounds=...:19000:0:99999:7:::
```

字段：
1. 用户名
2. 加密的密码
3. 上次修改密码的日期
4. 密码可更改的最小天数
5. 密码必须更改的最大天数
6. 密码过期警告天数
7. 密码过期后账户被禁用的天数
8. 账户过期日期

### `/etc/group` - 组信息
```
groupname:x:GID:user_list
```

示例：
```bash
sudo:x:27:zhang,li
developers:x:1001:zhang,wang,li
```

### `/etc/gshadow` - 组密码文件

## 4.3 用户管理命令

### `useradd` - 创建用户
```bash
# 基本创建
sudo useradd newuser

# 完整创建（推荐）
sudo useradd -m -s /bin/bash newuser
# -m: 创建主目录
# -s: 指定shell

# 更多选项
sudo useradd -m -s /bin/bash -c "Li Si" -G sudo,developers lisi
# -c: 注释
# -G: 附加组
# -d: 指定主目录
# -e: 账户过期日期
# -u: 指定UID

# 示例：创建系统用户（运行服务）
sudo useradd -r -s /usr/sbin/nologin -d /var/lib/myservice myservice
# -r: 创建系统用户
```

### `adduser` - 交互式创建用户（Debian/Ubuntu推荐）
```bash
sudo adduser newuser
# 会提示输入密码、全名等信息，并自动创建主目录
```

### `userdel` - 删除用户
```bash
sudo userdel username          # 删除用户（保留主目录）
sudo userdel -r username       # 删除用户和主目录
```

### `usermod` - 修改用户
```bash
# 添加到附加组
sudo usermod -aG sudo username      # -a: append, -G: groups

# 更改shell
sudo usermod -s /bin/zsh username

# 更改主目录
sudo usermod -d /new/home username

# 锁定账户
sudo usermod -L username            # Lock

# 解锁账户
sudo usermod -U username            # Unlock

# 更改用户名
sudo usermod -l newname oldname
```

### `passwd` - 密码管理
```bash
passwd                         # 修改自己的密码
sudo passwd username           # 修改其他用户密码
sudo passwd -l username        # 锁定用户
sudo passwd -u username        # 解锁用户
sudo passwd -d username        # 删除密码（不安全！）
sudo passwd -e username        # 使密码过期，强制用户下次登录时修改
```

## 4.4 组管理

### `groupadd` - 创建组
```bash
sudo groupadd developers       # 创建组
sudo groupadd -g 1500 testgroup  # 指定GID
```

### `groupdel` - 删除组
```bash
sudo groupdel groupname
```

### `groupmod` - 修改组
```bash
sudo groupmod -n newname oldname  # 重命名组
sudo groupmod -g 1600 groupname   # 更改GID
```

### `gpasswd` - 组成员管理
```bash
sudo gpasswd -a username groupname    # 添加用户到组
sudo gpasswd -d username groupname    # 从组删除用户
sudo gpasswd -A admin_user groupname  # 设置组管理员
```

### `groups` - 查看用户所属组
```bash
groups                         # 当前用户的组
groups username                # 指定用户的组
```

### `newgrp` - 临时切换主组
```bash
newgrp developers              # 切换到developers组
```

## 4.5 文件权限详解

### 权限表示

使用`ls -l`查看：
```bash
$ ls -l file.txt
-rw-r--r-- 1 zhang developers 1234 Jan 07 10:00 file.txt
```

解析权限字符串 `-rw-r--r--`：

```
- rw- r-- r--
│ │   │   │
│ │   │   └── 其他用户权限 (others)
│ │   └────── 组权限 (group)
│ └────────── 所有者权限 (user/owner)
└──────────── 文件类型
```

### 文件类型
- `-` 普通文件
- `d` 目录
- `l` 符号链接
- `c` 字符设备
- `b` 块设备
- `s` 套接字
- `p` 管道

### 权限含义

| 权限 | 文件 | 目录 | 八进制 |
|------|------|------|--------|
| `r` (read) | 读取文件内容 | 列出目录内容 | 4 |
| `w` (write) | 修改文件内容 | 在目录中创建/删除文件 | 2 |
| `x` (execute) | 执行文件 | 进入目录 | 1 |

**目录权限特别说明**：
- 需要`x`权限才能`cd`进入目录
- 需要`r+x`才能`ls`目录
- 需要`w+x`才能在目录中创建/删除文件

### 权限数字表示

```
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2 + 0 = 6
r-x = 4 + 0 + 1 = 5
r-- = 4 + 0 + 0 = 4
--- = 0 + 0 + 0 = 0
```

常见权限组合：
```
755 = rwxr-xr-x  (可执行文件、目录)
644 = rw-r--r--  (普通文件)
600 = rw-------  (私密文件)
700 = rwx------  (私密目录)
666 = rw-rw-rw-  (所有人可读写)
777 = rwxrwxrwx  (所有权限，危险！)
```

## 4.6 修改权限

### `chmod` - 修改文件权限

**符号模式**：
```bash
chmod u+x file.sh              # 所有者添加执行权限
chmod g-w file.txt             # 组去除写权限
chmod o+r file.txt             # 其他人添加读权限
chmod a+x file.sh              # 所有人添加执行权限 (all)
chmod u=rw,g=r,o=r file.txt    # 精确设置

# 组合
chmod u+x,g-w,o-r file.txt
```

符号：
- `u` - user (所有者)
- `g` - group (组)
- `o` - others (其他)
- `a` - all (所有)
- `+` - 添加
- `-` - 移除
- `=` - 精确设置

**数字模式**：
```bash
chmod 755 file.sh              # rwxr-xr-x
chmod 644 file.txt             # rw-r--r--
chmod 600 private.txt          # rw-------
chmod 700 script.sh            # rwx------

# 递归修改
chmod -R 755 directory/
```

### `chown` - 修改所有者
```bash
sudo chown zhang file.txt              # 只改所有者
sudo chown zhang:developers file.txt   # 改所有者和组
sudo chown :developers file.txt        # 只改组
sudo chown -R zhang:zhang directory/   # 递归修改
```

### `chgrp` - 修改所属组
```bash
sudo chgrp developers file.txt
sudo chgrp -R developers directory/
```

## 4.7 特殊权限

### SUID (Set User ID) - 4000
文件执行时，以文件所有者身份运行

```bash
chmod u+s file
chmod 4755 file

# 示例：/usr/bin/passwd
ls -l /usr/bin/passwd
# -rwsr-xr-x  （注意s位）
# 普通用户执行passwd时，临时获得root权限来修改/etc/shadow
```

### SGID (Set Group ID) - 2000
**对于文件**：执行时以文件所属组身份运行
**对于目录**：目录内新建文件继承目录的组

```bash
chmod g+s directory
chmod 2755 directory

# 示例：共享目录
sudo mkdir /shared
sudo chgrp developers /shared
sudo chmod 2775 /shared
# 现在developers组成员创建的文件都属于developers组
```

### Sticky Bit - 1000
只有文件所有者能删除自己的文件（即使目录是可写的）

```bash
chmod +t directory
chmod 1777 directory

# 示例：/tmp
ls -ld /tmp
# drwxrwxrwt  （注意最后的t）
# 所有人可以在/tmp创建文件，但只能删除自己的文件
```

### 特殊权限表示
```
SUID: s (有x) 或 S (无x) 在所有者执行位
SGID: s (有x) 或 S (无x) 在组执行位
Sticky: t (有x) 或 T (无x) 在其他执行位
```

## 4.8 默认权限 - umask

`umask`定义新建文件/目录的默认权限

```bash
umask                  # 查看当前umask (通常是0022)

# 计算实际权限
# 文件最大权限：666 (rw-rw-rw-)
# 目录最大权限：777 (rwxrwxrwx)
# 实际权限 = 最大权限 - umask

# umask = 0022
# 新建文件：666 - 022 = 644 (rw-r--r--)
# 新建目录：777 - 022 = 755 (rwxr-xr-x)

# 设置umask
umask 0077             # 只有所有者有权限
# 新建文件：600, 新建目录：700

# 永久设置（添加到~/.bashrc）
echo "umask 0077" >> ~/.bashrc
```

## 4.9 访问控制列表 (ACL)

ACL提供比传统Unix权限更细粒度的控制

### 查看ACL
```bash
getfacl file.txt
```

### 设置ACL
```bash
# 给特定用户权限
setfacl -m u:username:rwx file.txt

# 给特定组权限
setfacl -m g:groupname:rx file.txt

# 删除ACL
setfacl -x u:username file.txt

# 递归设置
setfacl -R -m u:username:rwx directory/

# 设置默认ACL（目录）
setfacl -d -m u:username:rwx directory/

# 删除所有ACL
setfacl -b file.txt
```

### 查看ACL
```bash
ls -l file.txt
# -rw-rwxr--+ 1 zhang developers 100 Jan 07 10:00 file.txt
#           ↑ 有+号表示有ACL
```

## 4.10 sudo - 临时提权

`sudo`允许普通用户以root权限执行命令

### 基本使用
```bash
sudo command                   # 以root身份执行
sudo -u username command       # 以指定用户身份执行
sudo -i                        # 切换到root shell
sudo -s                        # 以root身份启动shell
sudo su -                      # 切换到root用户（不推荐）
```

### 配置sudo权限

编辑配置文件（**必须**使用visudo）：
```bash
sudo visudo
```

`/etc/sudoers`语法：
```bash
# 允许zhang执行所有命令
zhang ALL=(ALL:ALL) ALL

# 允许developers组执行所有命令
%developers ALL=(ALL:ALL) ALL

# 允许zhang执行特定命令，无需密码
zhang ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# 允许zhang执行所有apt命令
zhang ALL=(ALL) /usr/bin/apt

# 别名
Cmnd_Alias SERVICES = /usr/bin/systemctl, /usr/sbin/service
User_Alias ADMINS = zhang, li
ADMINS ALL=(ALL) SERVICES
```

**重要**：
- 永远使用`visudo`编辑，它会检查语法错误
- 直接编辑/etc/sudoers可能导致sudo无法使用！

### sudo日志
查看sudo使用记录：
```bash
grep sudo /var/log/auth.log    # Debian/Ubuntu
grep sudo /var/log/secure       # RHEL/CentOS
```

## 4.11 实践练习

### 练习1：用户管理
```bash
# 创建用户
sudo adduser testuser

# 查看用户信息
id testuser
grep testuser /etc/passwd

# 添加到sudo组
sudo usermod -aG sudo testuser

# 切换到该用户
su - testuser

# 删除用户
sudo userdel -r testuser
```

### 练习2：权限实验
```bash
# 创建测试环境
mkdir ~/perm_test
cd ~/perm_test

# 创建文件
echo "secret" > secret.txt
echo "public" > public.txt

# 设置不同权限
chmod 600 secret.txt      # 只有所有者可读写
chmod 644 public.txt      # 所有人可读

# 创建可执行脚本
cat > script.sh << 'EOF'
#!/bin/bash
echo "Hello from script"
EOF

chmod 755 script.sh       # 可执行

# 查看权限
ls -l

# 测试执行
./script.sh
```

### 练习3：组权限协作
```bash
# 创建共享项目目录
sudo mkdir /opt/project
sudo groupadd developers
sudo chgrp developers /opt/project
sudo chmod 2775 /opt/project  # SGID + 775

# 添加用户到组
sudo usermod -aG developers $USER

# 新登录使组生效（或使用newgrp）
newgrp developers

# 测试：创建文件
touch /opt/project/testfile
ls -l /opt/project/testfile
# 应该属于developers组
```

### 练习4：ACL高级权限
```bash
# 创建文件
echo "data" > data.txt
chmod 600 data.txt

# 给特定用户只读权限
setfacl -m u:testuser:r data.txt

# 查看ACL
getfacl data.txt

# 测试：切换到testuser应该能读但不能写
```

### 练习5：sudo配置
```bash
# 查看sudo权限
sudo -l

# 编辑sudoers（练习环境）
sudo visudo -f /etc/sudoers.d/myconfig

# 添加：
# username ALL=(ALL) NOPASSWD: /usr/bin/apt update

# 测试
sudo apt update    # 不需要密码
```

## 4.12 安全最佳实践

### 1. 最小权限原则
```bash
# 不要
chmod 777 file      # 给所有人所有权限

# 应该
chmod 644 file      # 给最小必要权限
```

### 2. 不要用root登录
```bash
# 使用sudo代替直接root登录
# 禁用root SSH登录（/etc/ssh/sshd_config）：
PermitRootLogin no
```

### 3. 定期检查权限
```bash
# 查找所有SUID文件
find / -perm -4000 -type f 2>/dev/null

# 查找777权限文件（危险）
find / -perm 777 -type f 2>/dev/null

# 查找没有所有者的文件
find / -nouser -o -nogroup 2>/dev/null
```

### 4. 安全的密码策略
```bash
# 安装密码质量检查
sudo apt install libpam-pwquality

# 配置密码策略（/etc/security/pwquality.conf）
minlen = 12        # 最小长度
minclass = 3       # 至少3种字符类型
maxrepeat = 2      # 最多重复字符
```

### 5. 监控用户活动
```bash
# 查看登录历史
last
lastlog

# 查看失败登录尝试
sudo lastb

# 查看当前登录用户
w
who
```

## 4.13 常见问题

### Q: sudo和su有什么区别？
**A**:
- `sudo` - 以root权限执行单条命令，需要输入自己的密码
- `su` - 切换到root用户，需要输入root密码

### Q: 为什么执行./script.sh提示"Permission denied"？
**A**: 脚本没有执行权限，运行：`chmod +x script.sh`

### Q: 修改了组，为什么还不生效？
**A**: 需要重新登录或使用`newgrp groupname`使组生效

### Q: 删除用户时，保留还是删除主目录？
**A**:
- 临时用户：`userdel -r` 删除所有数据
- 重要用户：`userdel` 保留数据备份

## 下一步

完成本章后，你应该：
- ✅ 理解Linux用户和组系统
- ✅ 掌握文件权限管理
- ✅ 会使用sudo提权
- ✅ 了解ACL和特殊权限
- ✅ 知道安全最佳实践

**准备好了吗？让我们进入[第5章：Shell脚本编程](../05-Shell脚本/README.md)，开始自动化之旅！**

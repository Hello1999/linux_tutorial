# 第3章：基本命令入门

## 3.1 命令行基础

### Shell是什么？
Shell是用户与Linux内核交互的接口，最常用的是**Bash** (Bourne Again Shell)。

### 命令基本格式
```bash
命令 [选项] [参数]
```

示例：
```bash
ls -l /home
│  │  │
│  │  └─ 参数（要操作的对象）
│  └─ 选项（修改命令行为）
└─ 命令
```

### 选项格式
```bash
# 短选项（单个字符）
ls -l
ls -a
ls -la    # 组合多个短选项

# 长选项（完整单词）
ls --all
ls --human-readable

# 带值的选项
tar -f archive.tar
tar --file=archive.tar
```

### 获取帮助
```bash
man ls          # 查看详细手册（Manual）
ls --help       # 快速帮助
info ls         # Info文档（比man更详细）
tldr ls         # 实用示例（需安装tldr）
whatis ls       # 简短描述
```

**导航man页面**：
- `空格` - 下一页
- `b` - 上一页
- `/关键词` - 搜索
- `n` - 下一个匹配
- `q` - 退出

## 3.2 文件与目录操作

### `pwd` - 显示当前目录
```bash
pwd
# /home/zhang/Documents

# 显示物理路径（不含符号链接）
pwd -P
```

### `cd` - 切换目录
```bash
cd /etc               # 绝对路径
cd Documents          # 相对路径
cd ~                  # 回到主目录
cd                    # 同上（不带参数）
cd -                  # 回到上一次的目录
cd ..                 # 上级目录
cd ../..              # 上两级
cd /                  # 根目录
```

### `ls` - 列出文件
```bash
ls                    # 列出当前目录
ls /etc               # 列出指定目录
ls -l                 # 详细列表（long）
ls -a                 # 显示隐藏文件（all）
ls -la                # 组合选项
ls -lh                # 人类可读的文件大小（human-readable）
ls -lt                # 按修改时间排序（time）
ls -lS                # 按文件大小排序（Size）
ls -lR                # 递归显示子目录（Recursive）
ls -ld /etc           # 只显示目录本身，不显示内容
ls -li                # 显示inode编号

# 实用组合
ls -lht               # 详细、可读大小、时间排序
ls -lhS               # 详细、可读大小、大小排序
ls -la | grep "^d"    # 只显示目录
```

**颜色含义**（默认配色）：
- 蓝色：目录
- 绿色：可执行文件
- 红色：压缩文件
- 青色：符号链接
- 白色：普通文件

### `mkdir` - 创建目录
```bash
mkdir mydir                    # 创建单个目录
mkdir dir1 dir2 dir3           # 创建多个目录
mkdir -p parent/child/subchild # 递归创建（parent）
mkdir -m 755 mydir             # 指定权限（mode）
```

### `rmdir` - 删除空目录
```bash
rmdir mydir              # 只能删除空目录
rmdir -p a/b/c           # 递归删除空目录
```

### `touch` - 创建空文件或更新时间戳
```bash
touch file.txt           # 创建空文件（如果不存在）
touch file1 file2 file3  # 创建多个文件
touch existing.txt       # 更新已存在文件的访问和修改时间
```

### `cp` - 复制文件和目录
```bash
cp source.txt dest.txt             # 复制文件
cp source.txt /path/to/dest/       # 复制到目录
cp file1 file2 file3 /dest/        # 复制多个文件
cp -r dir1 dir2                    # 递归复制目录（recursive）
cp -i source dest                  # 覆盖前提示（interactive）
cp -u source dest                  # 只复制更新的文件（update）
cp -p source dest                  # 保留属性（权限、时间等）（preserve）
cp -a source dest                  # 归档模式（等于-dpR）

# 实用示例
cp -rp /source /backup             # 保留属性递归复制
cp -v source dest                  # 显示详细过程（verbose）
```

### `mv` - 移动或重命名
```bash
mv old.txt new.txt                 # 重命名文件
mv file.txt /path/to/dest/         # 移动文件
mv file1 file2 dir/                # 移动多个文件
mv -i source dest                  # 覆盖前提示
mv -n source dest                  # 不覆盖已存在文件（no-clobber）
mv -u source dest                  # 只移动更新的文件
mv -v source dest                  # 显示详细过程
```

### `rm` - 删除文件和目录
```bash
rm file.txt                        # 删除文件
rm file1 file2 file3               # 删除多个文件
rm -i file.txt                     # 删除前确认
rm -f file.txt                     # 强制删除，不提示（force）
rm -r directory                    # 递归删除目录（危险！）
rm -rf directory                   # 强制递归删除（非常危险！）
rm -v file.txt                     # 显示删除过程

# 安全删除
rm -i *                            # 删除前逐个确认
rm -I dir/*                        # 删除超过3个文件时提示一次

# 危险操作警告！
# sudo rm -rf / --no-preserve-root  # 删除整个系统（绝对不要运行！）
```

**安全建议**：
1. 删除前先用`ls`确认文件列表
2. 重要文件删除前备份
3. 考虑使用`trash-cli`工具（有回收站功能）

## 3.3 查看文件内容

### `cat` - 显示文件内容
```bash
cat file.txt                       # 显示文件内容
cat file1.txt file2.txt            # 显示多个文件
cat -n file.txt                    # 显示行号（number）
cat -b file.txt                    # 只给非空行编号
cat -s file.txt                    # 压缩连续空行（squeeze）
cat -A file.txt                    # 显示所有控制字符

# 创建文件
cat > newfile.txt                  # 输入内容，Ctrl+D结束
cat >> file.txt                    # 追加内容

# 合并文件
cat file1 file2 > merged.txt
```

### `less` - 分页查看（推荐）
```bash
less file.txt
```

**less快捷键**：
- `空格` / `f` - 下一页
- `b` - 上一页
- `g` - 文件开头
- `G` - 文件末尾
- `/pattern` - 向下搜索
- `?pattern` - 向上搜索
- `n` - 下一个匹配
- `N` - 上一个匹配
- `q` - 退出

### `more` - 分页查看（旧版，功能少）
```bash
more file.txt
```

### `head` - 查看文件开头
```bash
head file.txt                      # 默认显示前10行
head -n 20 file.txt                # 显示前20行
head -n 5 file1 file2              # 多个文件
head -c 100 file.txt               # 显示前100字节
```

### `tail` - 查看文件末尾
```bash
tail file.txt                      # 默认显示后10行
tail -n 20 file.txt                # 显示后20行
tail -f /var/log/syslog            # 实时跟踪文件更新（follow）
tail -F /var/log/syslog            # 文件重建后继续跟踪
tail -n +10 file.txt               # 从第10行开始显示

# 实用：查看日志
tail -f /var/log/apache2/access.log
```

### `wc` - 统计字数
```bash
wc file.txt                        # 显示 行数 单词数 字节数
wc -l file.txt                     # 只显示行数（lines）
wc -w file.txt                     # 只显示单词数（words）
wc -c file.txt                     # 只显示字节数（bytes）
wc -m file.txt                     # 显示字符数
```

## 3.4 文件搜索

### `find` - 强大的文件查找工具
```bash
# 按名称查找
find /path -name "*.txt"           # 查找txt文件
find /path -iname "*.TXT"          # 忽略大小写
find . -name "file*"               # 当前目录查找

# 按类型查找
find /path -type f                 # 文件
find /path -type d                 # 目录
find /path -type l                 # 符号链接

# 按大小查找
find /path -size +100M             # 大于100MB
find /path -size -10k              # 小于10KB
find /path -size 1G                # 等于1GB

# 按时间查找
find /path -mtime -7               # 7天内修改过
find /path -mtime +30              # 30天前修改
find /path -atime -1               # 1天内访问过

# 按权限查找
find /path -perm 644               # 权限为644的文件
find /path -perm -u+w              # 所有者可写的文件

# 组合条件
find /path -name "*.log" -size +10M
find /path -type f -name "*.tmp" -mtime +7 -delete

# 执行操作
find /path -name "*.txt" -exec cat {} \;
find /path -type f -exec chmod 644 {} \;
find . -name "*.bak" -ok rm {} \;  # 删除前确认

# 实用示例
# 查找当前目录下的大文件
find . -type f -size +100M -exec ls -lh {} \; | awk '{print $9 ": " $5}'

# 查找并删除空目录
find /path -type d -empty -delete

# 查找所有Python文件并统计行数
find . -name "*.py" -exec wc -l {} + | sort -n
```

### `locate` - 快速查找（使用数据库）
```bash
locate file.txt                    # 快速查找
locate -i file.txt                 # 忽略大小写
locate -n 10 "*.conf"              # 限制结果数量

# 更新数据库
sudo updatedb
```

**locate vs find**：
- `locate`：快速，但数据库可能不是最新
- `find`：实时搜索，功能强大，但速度慢

### `which` - 查找命令位置
```bash
which ls                           # /usr/bin/ls
which python3                      # /usr/bin/python3
```

### `whereis` - 查找二进制、源码、手册
```bash
whereis ls
# ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

## 3.5 文本处理基础

### `grep` - 文本搜索
```bash
grep "pattern" file.txt            # 搜索包含pattern的行
grep -i "pattern" file             # 忽略大小写（ignore-case）
grep -v "pattern" file             # 反向匹配（invert）
grep -n "pattern" file             # 显示行号（line-number）
grep -r "pattern" /path            # 递归搜索目录（recursive）
grep -l "pattern" *.txt            # 只显示文件名（files-with-matches）
grep -c "pattern" file             # 统计匹配行数（count）
grep -w "word" file                # 精确匹配单词（word-regexp）
grep -A 3 "pattern" file           # 显示匹配行及后3行（After）
grep -B 3 "pattern" file           # 显示匹配行及前3行（Before）
grep -C 3 "pattern" file           # 显示匹配行及前后3行（Context）

# 正则表达式
grep "^start" file                 # 以start开头的行
grep "end$" file                   # 以end结尾的行
grep "a.b" file                    # a和b之间有任意字符
grep "a.*b" file                   # a和b之间有任意字符（零个或多个）
grep "[0-9]" file                  # 包含数字的行
grep -E "pattern1|pattern2" file   # 扩展正则（或）

# 实用示例
# 查找所有错误日志
grep -i "error" /var/log/syslog

# 递归搜索代码中的TODO
grep -rn "TODO" ~/projects

# 排除二进制文件
grep -rI "pattern" /path

# 组合使用
ps aux | grep nginx
cat file.txt | grep "keyword" | wc -l
```

### `sort` - 排序
```bash
sort file.txt                      # 按字母排序
sort -r file.txt                   # 反向排序（reverse）
sort -n file.txt                   # 按数字排序（numeric）
sort -k 2 file.txt                 # 按第2列排序
sort -u file.txt                   # 排序并去重（unique）
sort -t: -k3 -n /etc/passwd        # 按:分隔，第3列，数字排序
```

### `uniq` - 去重（需先排序）
```bash
sort file.txt | uniq               # 去除相邻重复行
sort file.txt | uniq -c            # 统计重复次数
sort file.txt | uniq -d            # 只显示重复行
sort file.txt | uniq -u            # 只显示不重复的行
```

### `cut` - 提取列
```bash
cut -d: -f1 /etc/passwd            # 以:分隔，提取第1列
cut -d: -f1,3 /etc/passwd          # 提取第1和第3列
cut -c1-10 file.txt                # 提取每行的1-10个字符
```

### `tr` - 字符转换
```bash
echo "hello" | tr 'a-z' 'A-Z'      # 小写转大写
echo "hello" | tr -d 'l'           # 删除字符l
echo "hello  world" | tr -s ' '    # 压缩重复空格
```

## 3.6 重定向和管道

### 标准流
- **stdin (0)** - 标准输入
- **stdout (1)** - 标准输出
- **stderr (2)** - 标准错误

### 输出重定向
```bash
command > file                     # 覆盖写入文件
command >> file                    # 追加到文件
command 2> error.log               # 错误输出重定向
command 2>&1                       # 错误输出重定向到标准输出
command &> file                    # 标准输出和错误都重定向到文件
command > output.txt 2>&1          # 同上（兼容性更好）

# 丢弃输出
command > /dev/null                # 丢弃标准输出
command 2> /dev/null               # 丢弃错误输出
command &> /dev/null               # 丢弃所有输出
```

### 输入重定向
```bash
command < file                     # 从文件读取输入
command << EOF                     # Here Document
This is input
EOF

# 示例：创建配置文件
cat > config.txt << EOF
host=localhost
port=3306
EOF
```

### 管道
将一个命令的输出作为另一个命令的输入
```bash
command1 | command2

# 示例
ls -l | grep "\.txt"               # 列出txt文件
cat file.txt | sort | uniq         # 排序去重
ps aux | grep nginx | wc -l        # 统计nginx进程数

# 多级管道
cat /var/log/syslog | grep "error" | wc -l | xargs echo "错误数："

# tee：同时输出到屏幕和文件
ls -l | tee output.txt | grep "txt"
```

## 3.7 压缩与解压

### `tar` - 打包（最常用）
```bash
# 创建压缩包
tar -czf archive.tar.gz /path      # 创建gzip压缩包
tar -cjf archive.tar.bz2 /path     # 创建bzip2压缩包
tar -cJf archive.tar.xz /path      # 创建xz压缩包

# 解压
tar -xzf archive.tar.gz            # 解压gzip
tar -xjf archive.tar.bz2           # 解压bzip2
tar -xJf archive.tar.xz            # 解压xz
tar -xf archive.tar.gz             # 自动检测格式（推荐）

# 选项
tar -xzf archive.tar.gz -C /path   # 解压到指定目录
tar -tzf archive.tar.gz            # 查看压缩包内容（不解压）
tar -xzf archive.tar.gz file.txt   # 只解压指定文件
tar -czf archive.tar.gz --exclude="*.log" /path  # 排除文件

# 选项记忆
# c - create（创建）
# x - extract（解压）
# z - gzip
# j - bzip2
# J - xz
# f - file（必须，指定文件名）
# v - verbose（显示详细过程）
# t - list（列出内容）
```

### `gzip / gunzip` - gzip压缩
```bash
gzip file.txt                      # 压缩（源文件删除）
gzip -k file.txt                   # 保留源文件（keep）
gunzip file.txt.gz                 # 解压
gzip -d file.txt.gz                # 解压（同上）
gzip -r directory                  # 递归压缩目录中的文件
```

### `zip / unzip`
```bash
zip archive.zip file1 file2        # 创建zip
zip -r archive.zip directory       # 递归压缩目录
unzip archive.zip                  # 解压
unzip -l archive.zip               # 查看内容
unzip archive.zip -d /path         # 解压到指定目录
```

## 3.8 系统信息

### `uname` - 系统信息
```bash
uname -a                           # 显示所有信息
uname -r                           # 内核版本
uname -m                           # 硬件架构（x86_64）
uname -n                           # 主机名
```

### `hostname` - 主机名
```bash
hostname                           # 显示主机名
hostname -I                        # 显示IP地址
```

### `uptime` - 运行时间和负载
```bash
uptime
# 21:30:15 up 10 days, 3:45, 2 users, load average: 0.15, 0.20, 0.18
```

### `who / w` - 登录用户
```bash
who                                # 谁在登录
w                                  # 谁在登录，在做什么
whoami                             # 当前用户名
```

### `date` - 日期时间
```bash
date                               # 当前日期时间
date "+%Y-%m-%d"                   # 格式化输出：2026-01-07
date "+%Y-%m-%d %H:%M:%S"          # 2026-01-07 21:30:15
date -s "2026-01-07 21:30"         # 设置时间（需root）
```

### `cal` - 日历
```bash
cal                                # 当前月日历
cal 2026                           # 2026年日历
cal 12 2025                        # 2025年12月
```

## 3.9 进程管理基础

### `ps` - 查看进程
```bash
ps                                 # 当前终端的进程
ps -ef                             # 所有进程（完整格式）
ps aux                             # 所有进程（BSD格式）
ps aux | grep nginx                # 查找特定进程
ps -u username                     # 特定用户的进程
ps -p 1234                         # 特定PID的进程
```

### `top` - 动态进程监控
```bash
top
```

**top快捷键**：
- `q` - 退出
- `h` - 帮助
- `k` - 杀死进程
- `M` - 按内存排序
- `P` - 按CPU排序
- `1` - 显示所有CPU核心

### `htop` - top的增强版（需安装）
```bash
sudo apt install htop
htop
```

### `kill` - 终止进程
```bash
kill 1234                          # 发送TERM信号
kill -9 1234                       # 强制杀死（SIGKILL）
kill -15 1234                      # 优雅终止（SIGTERM，默认）
killall process_name               # 杀死所有同名进程
pkill -f pattern                   # 按模式杀死进程
```

**常用信号**：
- `SIGTERM (15)` - 请求终止（可以被捕获）
- `SIGKILL (9)` - 强制杀死（无法被捕获）
- `SIGHUP (1)` - 挂起
- `SIGINT (2)` - 中断（Ctrl+C）

### `bg / fg` - 后台/前台
```bash
command &                          # 在后台运行
jobs                               # 查看后台任务
fg %1                              # 将任务1调到前台
bg %1                              # 将任务1继续在后台运行
Ctrl+Z                             # 暂停当前任务
```

## 3.10 网络基础命令

### `ping` - 测试连通性
```bash
ping www.google.com                # 持续ping
ping -c 4 www.google.com           # ping 4次
ping -i 0.5 www.google.com         # 每0.5秒一次
```

### `ip` - 网络配置（现代）
```bash
ip addr                            # 显示IP地址
ip link                            # 显示网络接口
ip route                           # 显示路由表
```

### `ifconfig` - 网络配置（旧版，但仍常用）
```bash
ifconfig                           # 显示所有网络接口
ifconfig eth0                      # 显示特定接口
```

### `curl` - HTTP客户端
```bash
curl https://example.com           # 获取网页
curl -o file.html https://...      # 保存到文件
curl -O https://.../file.zip       # 使用远程文件名
curl -I https://example.com        # 只获取响应头
curl -L https://example.com        # 跟随重定向
```

### `wget` - 下载工具
```bash
wget https://example.com/file.zip  # 下载文件
wget -c https://...                # 断点续传
wget -r -np -k https://...         # 镜像网站
```

## 3.11 实践练习

### 练习1：文件操作综合
```bash
# 创建测试目录结构
mkdir -p ~/practice/dir1/subdir
mkdir -p ~/practice/dir2
cd ~/practice

# 创建文件
touch dir1/file1.txt
echo "Hello Linux" > dir1/file2.txt
cp dir1/file2.txt dir2/

# 移动和重命名
mv dir1/file1.txt dir2/renamed.txt

# 查看
ls -R
```

### 练习2：文本处理
```bash
# 创建测试数据
cat > users.txt << EOF
alice:1000:developer
bob:1001:designer
charlie:1002:developer
david:1003:manager
EOF

# 提取所有用户名
cut -d: -f1 users.txt

# 查找developer
grep "developer" users.txt

# 排序
sort users.txt

# 统计行数
wc -l users.txt
```

### 练习3：管道和重定向
```bash
# 查找当前目录最大的5个文件
du -sh * | sort -hr | head -5

# 统计代码行数
find . -name "*.sh" -exec wc -l {} + | sort -n

# 保存进程列表
ps aux > processes.txt
```

### 练习4：压缩实战
```bash
# 创建测试文件
mkdir backup
cd backup
touch file{1..10}.txt

# 打包压缩
tar -czf backup.tar.gz *

# 查看内容
tar -tzf backup.tar.gz

# 解压到新目录
mkdir restore
tar -xzf backup.tar.gz -C restore
```

## 3.12 命令技巧

### Tab补全
```bash
cd /etc/net[Tab]              # 自动补全network/
ls /usr/sh[Tab][Tab]          # 显示所有匹配项
```

### 历史命令
```bash
history                       # 查看命令历史
!100                          # 执行第100条命令
!!                            # 执行上一条命令
!ping                         # 执行最近的ping命令
Ctrl+R                        # 反向搜索历史
```

### 快捷键
```bash
Ctrl+C                        # 中断当前命令
Ctrl+Z                        # 暂停当前命令
Ctrl+D                        # 退出当前Shell（或EOF）
Ctrl+L                        # 清屏（等同于clear）
Ctrl+A                        # 光标移到行首
Ctrl+E                        # 光标移到行尾
Ctrl+U                        # 删除光标前的内容
Ctrl+K                        # 删除光标后的内容
Ctrl+W                        # 删除光标前的单词
Alt+.                         # 插入上一条命令的最后参数
```

### 命令别名
```bash
alias ll='ls -lh'
alias la='ls -lah'
alias ..='cd ..'
alias grep='grep --color=auto'

# 查看所有别名
alias

# 删除别名
unalias ll

# 永久保存（添加到~/.bashrc）
echo "alias ll='ls -lh'" >> ~/.bashrc
source ~/.bashrc
```

## 下一步

完成本章后，你应该：
- ✅ 掌握基本的文件操作命令
- ✅ 会查看和搜索文件内容
- ✅ 理解重定向和管道
- ✅ 能够压缩解压文件
- ✅ 了解基本的进程和网络命令

**准备好了吗？让我们进入[第4章：用户和权限管理](../04-用户权限/README.md)！**

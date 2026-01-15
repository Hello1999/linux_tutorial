# 第5章：Shell脚本编程

## 5.1 什么是Shell脚本？

Shell脚本是包含一系列命令的文本文件，用于自动化重复性任务。

### 第一个Shell脚本

创建`hello.sh`：
```bash
#!/bin/bash
# 这是注释

echo "Hello, World!"
echo "当前用户: $USER"
echo "当前目录: $(pwd)"
```

**Shebang (#!)** - 指定脚本解释器
```bash
#!/bin/bash        # Bash
#!/bin/sh          # POSIX sh
#!/usr/bin/env python3  # Python
```

### 执行脚本

```bash
# 方法1：添加执行权限
chmod +x hello.sh
./hello.sh

# 方法2：直接用bash执行
bash hello.sh

# 方法3：source执行（在当前shell中）
source hello.sh
. hello.sh         # 等同于source
```

## 5.2 变量

### 定义和使用变量

```bash
#!/bin/bash

# 定义变量（注意：=两边不能有空格！）
name="Zhang San"
age=25
city="Beijing"

# 使用变量
echo "姓名: $name"
echo "年龄: $age"
echo "城市: ${city}"     # 推荐用花括号

# 变量拼接
full_info="${name}_${age}_${city}"
echo $full_info
```

### 只读变量和删除变量

```bash
readonly PI=3.14159
# PI=3.14        # 报错：只读变量

unset name       # 删除变量
```

### 特殊变量

```bash
$0      # 脚本名称
$1-$9   # 位置参数
$#      # 参数个数
$*      # 所有参数（作为一个字符串）
$@      # 所有参数（作为独立字符串）
$?      # 上一条命令的退出状态
$$      # 当前进程PID
$!      # 最后一个后台进程PID
```

示例：
```bash
#!/bin/bash
# 文件名：args.sh

echo "脚本名: $0"
echo "第一个参数: $1"
echo "第二个参数: $2"
echo "参数总数: $#"
echo "所有参数: $@"
echo "脚本PID: $$"

# 运行：./args.sh hello world
```

### 命令替换

```bash
# 方法1：反引号（旧）
files=`ls /etc`

# 方法2：$() （推荐）
current_date=$(date +%Y-%m-%d)
file_count=$(ls | wc -l)

echo "今天是: $current_date"
echo "文件数: $file_count"
```

### 算术运算

```bash
# 方法1：$(())
a=10
b=20
sum=$((a + b))
echo "和: $sum"

# 支持的运算
echo $((10 + 5))    # 加
echo $((10 - 5))    # 减
echo $((10 * 5))    # 乘
echo $((10 / 5))    # 除
echo $((10 % 3))    # 取模
echo $((2 ** 3))    # 幂运算

# 自增自减
i=0
((i++))
echo $i             # 1
((i+=5))
echo $i             # 6

# 方法2：let
let result=10+5
echo $result

# 方法3：expr（旧）
result=`expr 10 + 5`
```

## 5.3 字符串操作

```bash
str="Hello World"

# 长度
echo ${#str}                    # 11

# 子字符串（从位置7开始，取5个字符）
echo ${str:6:5}                 # World

# 查找子字符串位置
echo `expr index "$str" "W"`    # 7

# 替换
echo ${str/World/Linux}         # Hello Linux（替换第一个）
echo ${str//o/O}                # HellO WOrld（替换所有）

# 删除
echo ${str#Hello }              # World（删除开头）
echo ${str%World}               # Hello （删除结尾）

# 大小写转换
upper="hello"
echo ${upper^^}                 # HELLO（全部大写）
echo ${upper^}                  # Hello（首字母大写）

lower="WORLD"
echo ${lower,,}                 # world（全部小写）
```

## 5.4 数组

```bash
# 定义数组
fruits=("apple" "banana" "orange")

# 另一种方式
colors[0]="red"
colors[1]="green"
colors[2]="blue"

# 访问元素
echo ${fruits[0]}               # apple
echo ${fruits[1]}               # banana

# 所有元素
echo ${fruits[@]}               # 所有元素
echo ${fruits[*]}               # 所有元素

# 数组长度
echo ${#fruits[@]}              # 3

# 遍历数组
for fruit in "${fruits[@]}"; do
    echo $fruit
done

# 添加元素
fruits+=("grape")

# 关联数组（字典）
declare -A person
person[name]="Zhang San"
person[age]=25
person[city]="Beijing"

echo ${person[name]}            # Zhang San
echo ${!person[@]}              # 所有键
echo ${person[@]}               # 所有值
```

## 5.5 条件判断

### if语句

```bash
#!/bin/bash

num=10

# 基本if
if [ $num -gt 5 ]; then
    echo "大于5"
fi

# if-else
if [ $num -lt 5 ]; then
    echo "小于5"
else
    echo "大于等于5"
fi

# if-elif-else
if [ $num -lt 5 ]; then
    echo "小于5"
elif [ $num -eq 5 ]; then
    echo "等于5"
else
    echo "大于5"
fi
```

### 测试命令

**数值比较**：
```bash
[ $a -eq $b ]    # 等于
[ $a -ne $b ]    # 不等于
[ $a -gt $b ]    # 大于
[ $a -lt $b ]    # 小于
[ $a -ge $b ]    # 大于等于
[ $a -le $b ]    # 小于等于
```

**字符串比较**：
```bash
[ "$a" = "$b" ]     # 相等
[ "$a" != "$b" ]    # 不相等
[ -z "$a" ]         # 字符串为空
[ -n "$a" ]         # 字符串非空
```

**文件测试**：
```bash
[ -e file ]      # 文件存在
[ -f file ]      # 是普通文件
[ -d dir ]       # 是目录
[ -r file ]      # 可读
[ -w file ]      # 可写
[ -x file ]      # 可执行
[ -s file ]      # 文件大小>0
[ file1 -nt file2 ]  # file1比file2新
[ file1 -ot file2 ]  # file1比file2旧
```

**逻辑运算**：
```bash
[ cond1 ] && [ cond2 ]   # 与
[ cond1 ] || [ cond2 ]   # 或
[ ! cond ]               # 非

# 组合
[ $a -gt 5 ] && [ $a -lt 10 ]
```

**现代测试：[[]]**（推荐）
```bash
# 支持正则匹配
if [[ "$str" =~ ^[0-9]+$ ]]; then
    echo "是数字"
fi

# 支持通配符
if [[ "$file" == *.txt ]]; then
    echo "是txt文件"
fi

# 逻辑运算更简洁
if [[ $a -gt 5 && $a -lt 10 ]]; then
    echo "在5到10之间"
fi
```

### case语句

```bash
#!/bin/bash

read -p "输入一个字符: " char

case $char in
    [a-z])
        echo "小写字母"
        ;;
    [A-Z])
        echo "大写字母"
        ;;
    [0-9])
        echo "数字"
        ;;
    *)
        echo "其他字符"
        ;;
esac
```

实用示例：
```bash
#!/bin/bash
# 服务管理脚本

case "$1" in
    start)
        echo "启动服务..."
        # 启动命令
        ;;
    stop)
        echo "停止服务..."
        # 停止命令
        ;;
    restart)
        echo "重启服务..."
        $0 stop
        $0 start
        ;;
    status)
        echo "查看状态..."
        # 状态命令
        ;;
    *)
        echo "用法: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

## 5.6 循环

### for循环

```bash
# 列表循环
for i in 1 2 3 4 5; do
    echo "数字: $i"
done

# 范围
for i in {1..10}; do
    echo $i
done

# 步长
for i in {0..20..2}; do  # 0到20，步长2
    echo $i
done

# C风格
for ((i=0; i<10; i++)); do
    echo $i
done

# 遍历文件
for file in *.txt; do
    echo "处理: $file"
done

# 遍历命令输出
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "用户: $user"
done
```

### while循环

```bash
# 基本while
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# 读取文件
while read line; do
    echo "行内容: $line"
done < file.txt

# 无限循环
while true; do
    echo "按Ctrl+C退出"
    sleep 1
done
```

### until循环

```bash
count=0
until [ $count -ge 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### 循环控制

```bash
# break - 退出循环
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo $i
done

# continue - 跳过本次循环
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue
    fi
    echo $i
done
```

## 5.7 函数

```bash
#!/bin/bash

# 定义函数
greet() {
    echo "Hello, $1!"
}

# 调用函数
greet "Zhang"

# 带返回值的函数
add() {
    local sum=$(($1 + $2))
    echo $sum
}

result=$(add 10 20)
echo "结果: $result"

# return返回状态码（0-255）
check_file() {
    if [ -f "$1" ]; then
        return 0  # 成功
    else
        return 1  # 失败
    fi
}

if check_file "/etc/passwd"; then
    echo "文件存在"
else
    echo "文件不存在"
fi

# 局部变量
test_vars() {
    local local_var="局部"
    global_var="全局"
    echo $local_var
}

test_vars
echo $global_var    # 全局变量可访问
# echo $local_var   # 局部变量不可访问
```

## 5.8 输入输出

### 读取用户输入

```bash
# 基本read
read -p "请输入名字: " name
echo "你好, $name"

# 读取多个变量
read -p "输入名字和年龄: " name age
echo "名字: $name, 年龄: $age"

# 限时输入
read -t 5 -p "5秒内输入: " input

# 密码输入（不显示）
read -s -p "输入密码: " password
echo

# 读取单个字符
read -n 1 -p "按任意键继续..."
echo
```

### printf格式化输出

```bash
# 比echo更强大
printf "姓名: %s, 年龄: %d\n" "Zhang" 25
printf "%-10s %-5d\n" "Name" "Age"
printf "%-10s %-5d\n" "Zhang" 25
printf "%-10s %-5d\n" "Li" 30

# 格式化数字
printf "%.2f\n" 3.14159  # 3.14
```

## 5.9 实用脚本示例

### 示例1：系统备份脚本

```bash
#!/bin/bash
# backup.sh - 备份脚本

# 配置
SOURCE_DIR="/home/user/documents"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"

# 检查源目录
if [ ! -d "$SOURCE_DIR" ]; then
    echo "错误: 源目录不存在"
    exit 1
fi

# 创建备份目录
mkdir -p "$BACKUP_DIR"

# 执行备份
echo "开始备份..."
tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "$SOURCE_DIR"

if [ $? -eq 0 ]; then
    echo "备份成功: ${BACKUP_FILE}"

    # 删除7天前的备份
    find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete
    echo "已清理旧备份"
else
    echo "备份失败"
    exit 1
fi
```

### 示例2：日志分析脚本

```bash
#!/bin/bash
# log_analyzer.sh - 分析日志文件

LOG_FILE="/var/log/syslog"

if [ ! -f "$LOG_FILE" ]; then
    echo "日志文件不存在"
    exit 1
fi

echo "=== 日志分析报告 ==="
echo "日志文件: $LOG_FILE"
echo "生成时间: $(date)"
echo ""

echo "总行数: $(wc -l < "$LOG_FILE")"
echo ""

echo "错误统计:"
grep -i "error" "$LOG_FILE" | wc -l
echo ""

echo "警告统计:"
grep -i "warning" "$LOG_FILE" | wc -l
echo ""

echo "最近10条错误:"
grep -i "error" "$LOG_FILE" | tail -10
```

### 示例3：批量重命名文件

```bash
#!/bin/bash
# rename_files.sh - 批量重命名

# 将所有.txt文件添加日期前缀

DATE_PREFIX=$(date +%Y%m%d)

for file in *.txt; do
    if [ -f "$file" ]; then
        new_name="${DATE_PREFIX}_${file}"
        mv "$file" "$new_name"
        echo "重命名: $file -> $new_name"
    fi
done

echo "完成！"
```

### 示例4：系统监控脚本

```bash
#!/bin/bash
# monitor.sh - 系统监控

echo "=== 系统监控报告 ==="
echo "时间: $(date)"
echo ""

# CPU使用率
echo "CPU使用率:"
top -bn1 | grep "Cpu(s)" | awk '{print "  " $2}'
echo ""

# 内存使用
echo "内存使用:"
free -h | awk 'NR==2{printf "  已用: %s / %s (%.2f%%)\n", $3, $2, $3/$2*100}'
echo ""

# 磁盘使用
echo "磁盘使用:"
df -h | grep -E '^/dev' | awk '{printf "  %s: %s / %s (%s)\n", $1, $3, $2, $5}'
echo ""

# 负载
echo "系统负载:"
uptime | awk -F'load average:' '{print "  " $2}'
echo ""

# 检查磁盘使用率告警
disk_usage=$(df -h | grep -E '^/dev' | awk '{print $5}' | sed 's/%//' | sort -nr | head -1)
if [ $disk_usage -gt 80 ]; then
    echo "警告: 磁盘使用率超过80%!"
fi
```

### 示例5：用户管理脚本

```bash
#!/bin/bash
# user_manager.sh - 用户管理工具

function show_menu() {
    echo "=== 用户管理工具 ==="
    echo "1. 创建用户"
    echo "2. 删除用户"
    echo "3. 列出所有用户"
    echo "4. 查看用户信息"
    echo "5. 退出"
    echo -n "请选择: "
}

function create_user() {
    read -p "输入用户名: " username
    sudo useradd -m -s /bin/bash "$username"
    sudo passwd "$username"
    echo "用户 $username 创建成功"
}

function delete_user() {
    read -p "输入要删除的用户名: " username
    read -p "是否删除主目录? (y/n): " del_home

    if [ "$del_home" = "y" ]; then
        sudo userdel -r "$username"
    else
        sudo userdel "$username"
    fi
    echo "用户 $username 已删除"
}

function list_users() {
    echo "系统用户列表:"
    cut -d: -f1 /etc/passwd | column
}

function user_info() {
    read -p "输入用户名: " username
    id "$username"
    echo "主目录: $(grep "^$username:" /etc/passwd | cut -d: -f6)"
    echo "Shell: $(grep "^$username:" /etc/passwd | cut -d: -f7)"
}

# 主循环
while true; do
    show_menu
    read choice

    case $choice in
        1) create_user ;;
        2) delete_user ;;
        3) list_users ;;
        4) user_info ;;
        5) echo "再见!"; exit 0 ;;
        *) echo "无效选择" ;;
    esac

    echo ""
    read -p "按回车继续..."
done
```

## 5.10 调试脚本

```bash
# 显示执行的每条命令
bash -x script.sh

# 在脚本中启用调试
#!/bin/bash
set -x          # 启用调试
# 代码
set +x          # 关闭调试

# 错误时退出
set -e          # 任何命令失败立即退出

# 未定义变量时退出
set -u

# 组合使用（推荐）
set -euo pipefail

# 调试输出
echo "DEBUG: 变量值 = $var" >&2
```

## 5.11 最佳实践

1. **始终使用shebang**
```bash
#!/bin/bash
```

2. **添加注释和说明**
```bash
#!/bin/bash
# 脚本名称：backup.sh
# 作者：Zhang San
# 日期：2026-01-07
# 描述：系统备份脚本
```

3. **检查命令是否成功**
```bash
if command; then
    echo "成功"
else
    echo "失败"
    exit 1
fi

# 或
command || { echo "失败"; exit 1; }
```

4. **使用函数组织代码**
5. **引用变量防止空格问题**
```bash
# 好
cp "$source_file" "$dest_dir"

# 不好
cp $source_file $dest_dir
```

6. **使用local声明函数内变量**
7. **提供使用说明**
```bash
if [ $# -lt 1 ]; then
    echo "用法: $0 <参数>"
    exit 1
fi
```

## 5.12 实践练习

### 练习1：创建用户信息脚本

编写一个脚本 `userinfo.sh`，实现以下功能：
- 接收用户名作为参数
- 显示该用户的详细信息（UID、GID、家目录、Shell）
- 如果用户不存在，显示错误信息

```bash
#!/bin/bash
# 提示：使用 id 命令和 getent passwd 命令
```

### 练习2：文件分类器

编写一个脚本 `classify_files.sh`：
- 遍历指定目录的所有文件
- 按照文件类型分类（文本文件、图片、视频、压缩包等）
- 统计每种类型的文件数量
- 输出统计结果

```bash
#!/bin/bash
# 提示：使用 file 命令判断文件类型
```

### 练习3：简易计算器

编写一个交互式计算器脚本 `calculator.sh`：
- 支持加减乘除运算
- 使用case语句处理不同运算符
- 进行错误检查（除数为0、非法输入等）
- 可以持续进行计算，输入q退出

```bash
#!/bin/bash
# 示例输出：
# 请输入第一个数字: 10
# 请选择运算符 (+, -, *, /): +
# 请输入第二个数字: 5
# 结果: 10 + 5 = 15
```

### 练习4：系统信息报告

编写一个脚本 `sysreport.sh`，生成系统信息报告：
- CPU使用率
- 内存使用情况
- 磁盘使用情况
- 运行时间（uptime）
- 当前登录用户
- 将报告保存到文件（包含时间戳）

```bash
#!/bin/bash
# 输出文件：report_YYYYMMDD_HHMMSS.txt
```

### 练习5：批处理工具

编写一个脚本 `batch_rename.sh`：
- 接收目录路径和模式作为参数
- 批量重命名文件（例如：添加前缀、替换字符等）
- 重命名前显示预览，询问用户确认
- 实现回滚功能（保存原始文件名）

```bash
#!/bin/bash
# 用法: ./batch_rename.sh /path/to/dir "prefix_"
```

### 练习6：日志轮换脚本

编写一个日志轮换脚本 `rotate_logs.sh`：
- 检查日志文件大小
- 如果超过指定大小（如10MB），进行轮换
- 保留最近N个日志文件
- 压缩旧日志
- 添加日志记录功能

```bash
#!/bin/bash
# 配置：
# MAX_SIZE=10M
# KEEP_DAYS=7
```

### 练习7：进程监控器

编写一个脚本 `process_monitor.sh`：
- 监控指定进程是否运行
- 如果进程不存在，自动重启
- 记录重启次数和时间
- 如果重启失败超过N次，发送告警（写入日志或发送邮件）

```bash
#!/bin/bash
# 用法: ./process_monitor.sh nginx
```

### 练习8：备份验证脚本

扩展之前的备份脚本，添加以下功能：
- 备份前检查磁盘空间是否足够
- 备份后验证文件完整性（MD5校验）
- 自动清理超过30天的旧备份
- 生成备份报告（成功/失败、文件大小、耗时等）
- 支持增量备份

### 挑战练习：多功能系统管理工具

编写一个综合脚本 `sysadmin.sh`，整合多个管理功能：
```bash
#!/bin/bash
# 系统管理工具菜单
# 1. 系统信息查看
# 2. 用户管理
# 3. 备份管理
# 4. 日志查看
# 5. 网络诊断
# 6. 退出

# 要求：
# - 使用函数组织代码
# - 菜单循环直到用户选择退出
# - 每个功能都要有错误处理
# - 添加日志记录
# - 支持命令行参数直接执行功能
```

### 调试技巧练习

找出以下脚本的问题并修复：

```bash
#!/bin/bash
# 这个脚本有多个错误，请找出并修复

count=0
for file in *.txt
do
    if [ -f $file ]
    then
        lines=$(wc -l < $file)
        count=$count+$lines
        echo "文件 $file 有 $lines 行"
    fi
done

echo "总共 $count 行"
```

提示：至少有5个问题需要修复

### 学习建议

完成这些练习后：
1. 对比不同实现方法的优劣
2. 使用 `shellcheck` 工具检查脚本质量
3. 阅读他人的优秀脚本，学习编程技巧
4. 尝试优化你的脚本（性能、可读性、健壮性）
5. 为常用任务编写自己的脚本工具库

## 下一步

完成本章后，你应该：
- ✅ 能编写基本Shell脚本
- ✅ 掌握变量、条件、循环
- ✅ 会编写函数
- ✅ 能够调试脚本
- ✅ 通过实践练习巩固了所学知识

**继续深入学习后续章节，或者查看[实战项目](../18-实战项目/README.md)应用所学知识！**

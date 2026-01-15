# 第17章：高级Shell编程

## 17.1 高级Shell特性

### Bash扩展

```bash
# 花括号扩展
echo {1..10}              # 1 2 3 4 5 6 7 8 9 10
echo {a..z}               # a b c ... z
echo file{1,2,3}.txt      # file1.txt file2.txt file3.txt
echo {A..Z}{0..9}         # A0 A1 ... Z9

# 参数扩展
name="John"
echo ${name}              # John
echo ${name:-"Default"}   # John（如果name为空则用"Default"）
echo ${name:="Default"}   # 如果name为空则赋值为"Default"
echo ${name:?"Error"}     # 如果name为空则显示错误并退出

# 字符串操作
str="Hello World"
echo ${str#Hello }        # World（删除开头）
echo ${str%World}         # Hello （删除结尾）
echo ${str/World/Linux}   # Hello Linux（替换）
echo ${str//o/O}          # HellO WOrld（替换所有）
echo ${#str}              # 11（长度）
echo ${str:0:5}           # Hello（子字符串）
echo ${str^^}             # HELLO WORLD（大写）
echo ${str,,}             # hello world（小写）

# 数组扩展
arr=(one two three)
echo ${arr[@]}            # 所有元素
echo ${arr[0]}            # 第一个元素
echo ${#arr[@]}           # 数组长度
echo ${arr[@]:1:2}        # 切片（从索引1开始，取2个元素）
```

### 进程替换

```bash
# 比较两个命令的输出
diff <(ls dir1) <(ls dir2)

# 将命令输出作为文件输入
while read line; do
    echo "Line: $line"
done < <(cat file.txt)

# 示例：比较两个服务器的文件
diff <(ssh server1 "ls /data") <(ssh server2 "ls /data")
```

### 协程（Coprocess）

```bash
# 创建协程
coproc myproc { while read line; do echo "Got: $line"; done; }

# 向协程写入
echo "hello" >&${myproc[1]}

# 从协程读取
read -u ${myproc[0]} response
echo $response
```

## 17.2 高级函数技巧

### 函数高级用法

```bash
# 递归函数
factorial() {
    if [ $1 -le 1 ]; then
        echo 1
    else
        local temp=$(factorial $(($1 - 1)))
        echo $(($1 * temp))
    fi
}

result=$(factorial 5)
echo "5! = $result"  # 120

# 返回数组
get_array() {
    local arr=("apple" "banana" "orange")
    echo "${arr[@]}"
}

fruits=($(get_array))
echo "第一个水果: ${fruits[0]}"

# 命名参数（使用关联数组）
process_args() {
    declare -A args
    while [[ $# -gt 0 ]]; do
        case $1 in
            --name)
                args[name]="$2"
                shift 2
                ;;
            --age)
                args[age]="$2"
                shift 2
                ;;
            *)
                shift
                ;;
        esac
    done

    echo "Name: ${args[name]}"
    echo "Age: ${args[age]}"
}

process_args --name "John" --age 30
```

### 函数库

```bash
# mylib.sh
#!/bin/bash

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"
}

error() {
    log "ERROR: $*" >&2
    return 1
}

success() {
    log "SUCCESS: $*"
    return 0
}

# main.sh
#!/bin/bash
source ./mylib.sh

log "开始处理..."
if some_command; then
    success "处理完成"
else
    error "处理失败"
fi
```

## 17.3 错误处理

### set选项

```bash
#!/bin/bash

# 遇到错误立即退出
set -e

# 管道中任何命令失败都退出
set -o pipefail

# 使用未定义变量时报错
set -u

# 打印执行的命令（调试）
set -x

# 组合使用
set -euo pipefail

# 示例
set -euo pipefail

command1
command2 | command3  # 如果任何命令失败，脚本退出
command4
```

### trap错误处理

```bash
#!/bin/bash

# 清理函数
cleanup() {
    echo "清理临时文件..."
    rm -f /tmp/tempfile
}

# 注册清理函数
trap cleanup EXIT

# 错误处理
error_handler() {
    echo "错误发生在第 $1 行"
    cleanup
    exit 1
}

trap 'error_handler $LINENO' ERR

# 主逻辑
echo "开始处理..."
some_command
echo "完成"
```

### 自定义错误处理

```bash
#!/bin/bash

# 错误处理框架
readonly LOG_FILE="/var/log/myscript.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

die() {
    log "FATAL: $*"
    exit 1
}

warn() {
    log "WARNING: $*"
}

# 检查命令是否存在
require_command() {
    command -v "$1" &>/dev/null || die "需要命令: $1"
}

# 检查文件是否存在
require_file() {
    [[ -f "$1" ]] || die "文件不存在: $1"
}

# 使用
require_command "docker"
require_file "/etc/config.conf"

if ! some_command; then
    warn "某个命令失败了"
fi
```

## 17.4 并行处理

### 使用后台进程

```bash
#!/bin/bash

# 简单并行
for i in {1..10}; do
    (
        echo "处理任务 $i"
        sleep 1
        echo "任务 $i 完成"
    ) &
done

wait  # 等待所有后台进程完成
echo "所有任务完成"
```

### 控制并发数

```bash
#!/bin/bash

MAX_JOBS=4
job_count=0

for i in {1..20}; do
    (
        echo "处理任务 $i"
        sleep 2
    ) &

    ((job_count++))

    # 达到最大并发数，等待
    if (( job_count >= MAX_JOBS )); then
        wait -n  # 等待任意一个任务完成
        ((job_count--))
    fi
done

wait  # 等待剩余任务
echo "所有任务完成"
```

### 使用GNU Parallel

```bash
# 安装
sudo apt install parallel

# 基本用法
seq 10 | parallel echo "处理任务 {}"

# 并行执行命令
parallel "echo {}; sleep 1" ::: {1..10}

# 从文件读取
cat urls.txt | parallel curl -O {}

# 控制并发数
seq 100 | parallel -j 4 "sleep 1; echo {}"

# 多个输入
parallel echo {1} {2} ::: A B C ::: 1 2 3

# 实用示例：并行压缩文件
find . -name "*.txt" | parallel gzip {}

# 并行下载
cat urls.txt | parallel -j 5 wget -q {}
```

## 17.5 交互式脚本

### 读取输入

```bash
# 基本读取
read -p "请输入名字: " name
echo "你好, $name"

# 密码输入（不显示）
read -s -p "请输入密码: " password
echo

# 超时读取
if read -t 5 -p "5秒内输入（按Enter继续）: " input; then
    echo "输入: $input"
else
    echo "超时！"
fi

# 读取单个字符
read -n 1 -p "按任意键继续..."
echo

# 读取到数组
read -a array -p "输入多个值（空格分隔）: "
echo "第一个值: ${array[0]}"
```

### 菜单系统

```bash
#!/bin/bash

show_menu() {
    clear
    echo "========================"
    echo "   系统管理菜单"
    echo "========================"
    echo "1. 查看系统信息"
    echo "2. 查看磁盘使用"
    echo "3. 查看内存使用"
    echo "4. 查看进程"
    echo "5. 退出"
    echo "========================"
}

while true; do
    show_menu
    read -p "请选择 [1-5]: " choice

    case $choice in
        1)
            echo "系统信息:"
            uname -a
            read -p "按Enter继续..."
            ;;
        2)
            echo "磁盘使用:"
            df -h
            read -p "按Enter继续..."
            ;;
        3)
            echo "内存使用:"
            free -h
            read -p "按Enter继续..."
            ;;
        4)
            echo "进程列表:"
            ps aux | head -20
            read -p "按Enter继续..."
            ;;
        5)
            echo "再见！"
            exit 0
            ;;
        *)
            echo "无效选择"
            read -p "按Enter继续..."
            ;;
    esac
done
```

### 进度条

```bash
# 简单进度条
progress_bar() {
    local duration=$1
    local steps=50
    local step_duration=$((duration * 1000 / steps))

    for ((i=0; i<=steps; i++)); do
        local percent=$((i * 100 / steps))
        local filled=$((i * 50 / steps))
        local empty=$((50 - filled))

        printf "\r[%${filled}s%${empty}s] %d%%" | tr ' ' '=' | tr ' ' ' '
        printf " " | tr ' ' '='
        printf "%d%%" "$percent"

        sleep 0.${step_duration}
    done
    echo
}

echo "处理中..."
progress_bar 3
echo "完成！"

# 使用pv（管道查看器）
sudo apt install pv
dd if=/dev/zero bs=1M count=100 2>&1 | pv -s 100M > /dev/null
```

## 17.6 调试技巧

### 调试选项

```bash
#!/bin/bash

# 开启调试模式
set -x   # 打印执行的命令
set -v   # 打印读取的命令

# 部分调试
set -x
# 需要调试的代码
set +x

# 使用PS4自定义调试输出
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
set -x
```

### 调试函数

```bash
# 调试输出函数
DEBUG=${DEBUG:-0}

debug() {
    if [[ $DEBUG -eq 1 ]]; then
        echo "[DEBUG] $*" >&2
    fi
}

# 使用
debug "变量值: $var"
debug "进入函数 process_data"

# 运行时启用调试
# DEBUG=1 ./script.sh
```

### shellcheck

```bash
# 安装shellcheck
sudo apt install shellcheck

# 检查脚本
shellcheck myscript.sh

# 忽略特定警告
# shellcheck disable=SC2086
variable=$filename
```

## 17.7 性能优化

### 避免子shell

```bash
# 慢（创建子shell）
cat file.txt | while read line; do
    echo $line
done

# 快（不创建子shell）
while read line; do
    echo $line
done < file.txt

# 慢（多次命令替换）
for i in {1..1000}; do
    result=$(date +%s)
done

# 快（一次性获取）
now=$(date +%s)
for i in {1..1000}; do
    result=$now
done
```

### 使用内置命令

```bash
# 慢（外部命令）
if [ "$(cat file.txt | wc -l)" -gt 100 ]; then
    echo "很多行"
fi

# 快（内置）
lines=0
while read; do
    ((lines++))
done < file.txt

if [ $lines -gt 100 ]; then
    echo "很多行"
fi
```

### 批量处理

```bash
# 慢（逐个处理）
for file in *.txt; do
    gzip "$file"
done

# 快（并行处理）
find . -name "*.txt" | parallel gzip {}

# 快（使用xargs）
find . -name "*.txt" -print0 | xargs -0 -P 4 gzip
```

## 17.8 实用高级脚本

### 脚本1：备份工具

```bash
#!/bin/bash
set -euo pipefail

# 配置
SOURCE="/data"
DEST="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"
MAX_BACKUPS=7

# 日志
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a backup.log
}

# 创建备份
log "开始备份 $SOURCE"
tar -czf "$DEST/$BACKUP_FILE" "$SOURCE" 2>&1 | tee -a backup.log

if [ ${PIPESTATUS[0]} -eq 0 ]; then
    log "备份成功: $BACKUP_FILE"

    # 清理旧备份
    log "清理旧备份..."
    cd "$DEST"
    ls -t backup_*.tar.gz | tail -n +$((MAX_BACKUPS + 1)) | xargs -r rm
    log "清理完成"
else
    log "备份失败"
    exit 1
fi
```

### 脚本2：系统监控

```bash
#!/bin/bash

# 配置
CPU_THRESHOLD=80
MEM_THRESHOLD=80
DISK_THRESHOLD=90

# 发送告警
send_alert() {
    local message=$1
    echo "$message" | mail -s "系统告警" admin@example.com
    # 或使用其他通知方式
}

# 检查CPU
check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    if (( $(echo "$cpu_usage > $CPU_THRESHOLD" | bc -l) )); then
        send_alert "CPU使用率过高: ${cpu_usage}%"
    fi
}

# 检查内存
check_memory() {
    local mem_usage=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
    if [ $mem_usage -gt $MEM_THRESHOLD ]; then
        send_alert "内存使用率过高: ${mem_usage}%"
    fi
}

# 检查磁盘
check_disk() {
    df -h | awk 'NR>1 && $5+0 > '$DISK_THRESHOLD' {
        print "磁盘使用率过高: "$6" "$5
    }' | while read line; do
        send_alert "$line"
    done
}

# 主循环
while true; do
    check_cpu
    check_memory
    check_disk
    sleep 60
done
```

### 脚本3：日志分析

```bash
#!/bin/bash

LOG_FILE="/var/log/apache2/access.log"
REPORT_FILE="report_$(date +%Y%m%d).txt"

{
    echo "=== 访问日志分析报告 ==="
    echo "生成时间: $(date)"
    echo ""

    echo "=== 总访问次数 ==="
    wc -l < "$LOG_FILE"
    echo ""

    echo "=== Top 10 访问IP ==="
    awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10
    echo ""

    echo "=== Top 10 访问页面 ==="
    awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10
    echo ""

    echo "=== 状态码分布 ==="
    awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn
    echo ""

    echo "=== 每小时访问量 ==="
    awk '{print $4}' "$LOG_FILE" | cut -d: -f2 | sort | uniq -c

} > "$REPORT_FILE"

echo "报告已生成: $REPORT_FILE"
```

## 17.9 脚本安全

### 输入验证

```bash
# 验证数字
is_number() {
    [[ $1 =~ ^[0-9]+$ ]]
}

read -p "输入数字: " num
if is_number "$num"; then
    echo "有效数字: $num"
else
    echo "无效输入"
    exit 1
fi

# 验证文件路径
is_safe_path() {
    [[ $1 == /* ]] || return 1  # 必须是绝对路径
    [[ $1 == *..* ]] && return 1  # 不含..
    return 0
}

# 验证邮箱
is_email() {
    [[ $1 =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}
```

### 安全实践

```bash
#!/bin/bash
set -euo pipefail

# 1. 引用所有变量
file="my file.txt"
rm "$file"  # 正确
# rm $file  # 错误（会被解释为两个文件）

# 2. 使用临时文件
TMPFILE=$(mktemp)
trap "rm -f $TMPFILE" EXIT

# 3. 检查命令是否存在
command -v docker &>/dev/null || { echo "需要docker"; exit 1; }

# 4. 避免命令注入
# 危险
user_input="test; rm -rf /"
eval "echo $user_input"  # 不要这样做！

# 安全
printf '%s\n' "$user_input"

# 5. 限制权限
umask 077  # 新文件只有所有者可读写

# 6. 使用专用工具
# 不要自己解析JSON，使用jq
result=$(curl -s api.com | jq -r '.data')
```

## 17.10 实践练习

### 练习1：实现计算器

```bash
#!/bin/bash

calc() {
    local expr="$*"
    bc -l <<< "$expr"
}

# 使用
calc "10 + 20"
calc "3.14 * 2"
calc "sqrt(16)"
```

### 练习2：文件监控

```bash
#!/bin/bash

watch_file() {
    local file=$1
    local last_mod=$(stat -c %Y "$file")

    while true; do
        local current_mod=$(stat -c %Y "$file")
        if [ $current_mod -ne $last_mod ]; then
            echo "文件已更改: $file"
            last_mod=$current_mod
        fi
        sleep 1
    done
}

watch_file "/etc/hosts"
```

### 练习3：实现简单的任务队列

```bash
#!/bin/bash

QUEUE_DIR="/tmp/taskqueue"
mkdir -p "$QUEUE_DIR"

# 添加任务
add_task() {
    local task=$1
    local id=$(date +%s%N)
    echo "$task" > "$QUEUE_DIR/$id"
}

# 处理任务
process_tasks() {
    while true; do
        for task_file in "$QUEUE_DIR"/*; do
            [ -f "$task_file" ] || continue

            local task=$(cat "$task_file")
            echo "处理任务: $task"
            eval "$task"
            rm "$task_file"
        done
        sleep 1
    done
}

# 使用
add_task "echo 'Task 1'"
add_task "sleep 2 && echo 'Task 2'"
process_tasks
```

## 下一步

完成本章后，你应该：
- ✅ 掌握高级Shell特性
- ✅ 会处理复杂的错误和异常
- ✅ 能够编写并行脚本
- ✅ 了解脚本调试和优化
- ✅ 掌握脚本安全最佳实践

**恭喜你完成了Linux学习的全部章节！继续实践，探索更多[实战项目](../18-实战项目/README.md)来巩固所学知识！**

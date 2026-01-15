# 第6章：文本处理工具

## 6.1 sed - 流编辑器

`sed` (Stream Editor) 是一个强大的文本处理工具，可以对文本进行搜索、替换、删除、插入等操作。

### 基本语法

```bash
sed [选项] '命令' 文件
sed [选项] -e '命令1' -e '命令2' 文件
sed [选项] -f 脚本文件 文件
```

### 常用选项

```bash
-n              # 静默模式，不自动打印模式空间
-e              # 执行多个sed命令
-f              # 从文件读取sed命令
-i              # 直接修改文件内容（危险！）
-i.bak          # 修改前备份
-r 或 -E        # 使用扩展正则表达式
```

### 替换操作 (s命令)

```bash
# 基本替换（只替换每行第一个匹配）
sed 's/old/new/' file.txt

# 全局替换（替换所有匹配）
sed 's/old/new/g' file.txt

# 只替换第2个匹配
sed 's/old/new/2' file.txt

# 忽略大小写
sed 's/old/new/gi' file.txt

# 使用不同分隔符（处理路径时很有用）
sed 's|/usr/local|/opt|g' file.txt
sed 's#/usr/local#/opt#g' file.txt

# 直接修改文件
sed -i 's/old/new/g' file.txt

# 修改前备份
sed -i.bak 's/old/new/g' file.txt
```

### 删除操作 (d命令)

```bash
# 删除第3行
sed '3d' file.txt

# 删除第3到第5行
sed '3,5d' file.txt

# 删除最后一行
sed '$d' file.txt

# 删除空行
sed '/^$/d' file.txt

# 删除包含pattern的行
sed '/pattern/d' file.txt

# 删除以#开头的行（注释行）
sed '/^#/d' file.txt
```

### 打印操作 (p命令)

```bash
# 打印第3行
sed -n '3p' file.txt

# 打印第3到第5行
sed -n '3,5p' file.txt

# 打印匹配行
sed -n '/pattern/p' file.txt

# 打印匹配行到文件末尾
sed -n '/pattern/,$p' file.txt
```

### 插入和追加 (i和a命令)

```bash
# 在第3行前插入文本
sed '3i\新插入的行' file.txt

# 在第3行后追加文本
sed '3a\新追加的行' file.txt

# 在匹配行前插入
sed '/pattern/i\新行' file.txt

# 在匹配行后追加
sed '/pattern/a\新行' file.txt

# 插入多行
sed '3i\第一行\n第二行\n第三行' file.txt
```

### 修改操作 (c命令)

```bash
# 替换整行
sed '3c\新的内容' file.txt

# 替换匹配的行
sed '/pattern/c\新的内容' file.txt
```

### 地址范围

```bash
# 第3行到第5行
sed '3,5s/old/new/g' file.txt

# 从匹配行到第10行
sed '/start/,10s/old/new/g' file.txt

# 从第5行到匹配行
sed '5,/end/s/old/new/g' file.txt

# 两个匹配之间
sed '/start/,/end/s/old/new/g' file.txt

# 从匹配行到末尾
sed '/pattern/,$s/old/new/g' file.txt
```

### 实用示例

```bash
# 删除HTML标签
sed 's/<[^>]*>//g' file.html

# 为每行添加行号
sed = file.txt | sed 'N;s/\n/\t/'

# 删除每行开头的空格
sed 's/^[ \t]*//' file.txt

# 删除每行末尾的空格
sed 's/[ \t]*$//' file.txt

# 在每行开头添加#
sed 's/^/#/' file.txt

# 注释掉包含pattern的行
sed '/pattern/s/^/#/' file.txt

# 取消注释
sed 's/^#//' file.txt

# 删除C/C++风格的注释
sed 's|//.*||' file.cpp

# 只保留IP地址列
sed -n 's/.*\([0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\).*/\1/p' file.txt
```

## 6.2 awk - 文本分析工具

`awk` 是一个强大的文本处理编程语言，特别适合处理列式数据。

### 基本语法

```bash
awk '模式 {动作}' 文件
awk -F: '{print $1}' /etc/passwd
```

### 内置变量

```bash
$0          # 整行内容
$1, $2...   # 第1列、第2列...
NF          # 当前行的字段数
NR          # 当前行号
FNR         # 当前文件的行号
FS          # 字段分隔符（默认空格）
OFS         # 输出字段分隔符
RS          # 记录分隔符（默认换行）
ORS         # 输出记录分隔符
FILENAME    # 当前文件名
```

### 基本用法

```bash
# 打印第1列
awk '{print $1}' file.txt

# 打印第1列和第3列
awk '{print $1, $3}' file.txt

# 打印整行
awk '{print $0}' file.txt
awk '{print}' file.txt  # 简写

# 指定分隔符
awk -F: '{print $1}' /etc/passwd
awk -F'[,:]' '{print $1}' file.txt  # 多个分隔符

# 多个命令
awk '{print $1} {print $2}' file.txt
awk '{print $1; print $2}' file.txt
```

### 模式匹配

```bash
# 打印包含pattern的行
awk '/pattern/' file.txt

# 打印第3列包含pattern的行
awk '$3 ~ /pattern/' file.txt

# 打印第3列不包含pattern的行
awk '$3 !~ /pattern/' file.txt

# 打印第3列等于value的行
awk '$3 == "value"' file.txt

# 条件判断
awk '$3 > 100' file.txt          # 第3列大于100
awk '$1 == "root"' /etc/passwd   # 第1列等于root
awk 'NR > 10' file.txt           # 行号大于10
```

### BEGIN和END

```bash
# BEGIN在处理文件前执行
awk 'BEGIN {print "开始处理"} {print $1} END {print "处理完成"}' file.txt

# 计算总数
awk 'BEGIN {sum=0} {sum+=$3} END {print "总和:", sum}' file.txt

# 统计行数
awk 'END {print NR}' file.txt
```

### 数学运算

```bash
# 计算第3列总和
awk '{sum += $3} END {print sum}' file.txt

# 计算平均值
awk '{sum += $3; count++} END {print sum/count}' file.txt

# 找出最大值
awk 'BEGIN {max=0} {if($3>max) max=$3} END {print max}' file.txt

# 多列运算
awk '{print $1, $2, $3+$4}' file.txt
```

### 格式化输出

```bash
# printf格式化
awk '{printf "%-10s %5d\n", $1, $2}' file.txt

# 对齐输出
awk '{printf "%10s %10s\n", $1, $2}' file.txt

# 格式化数字
awk '{printf "%.2f\n", $3}' file.txt  # 保留2位小数
```

### 条件和循环

```bash
# if语句
awk '{if($3>100) print $1}' file.txt

# if-else
awk '{if($3>100) print "高"; else print "低"}' file.txt

# for循环
awk '{for(i=1;i<=NF;i++) print $i}' file.txt

# while循环
awk '{i=1; while(i<=NF) {print $i; i++}}' file.txt
```

### 数组

```bash
# 统计每个值出现的次数
awk '{count[$1]++} END {for(i in count) print i, count[i]}' file.txt

# 去重
awk '!seen[$1]++' file.txt

# 统计单词频率
awk '{for(i=1;i<=NF;i++) count[$i]++} END {for(w in count) print w, count[w]}' file.txt
```

### 实用示例

```bash
# 打印第1列和最后一列
awk '{print $1, $NF}' file.txt

# 反转列顺序
awk '{for(i=NF;i>0;i--) printf "%s ", $i; print ""}' file.txt

# 统计文件大小总和（ls -l输出）
ls -l | awk '{sum += $5} END {print "总大小:", sum}'

# 查看内存使用
free -m | awk 'NR==2{printf "内存使用率: %.2f%%\n", $3/$2*100}'

# 分析日志文件
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# 处理CSV文件
awk -F, '{print $2, $4}' data.csv

# 计算平均响应时间
awk '{sum+=$10; count++} END {print "平均:", sum/count "ms"}' access.log

# 按条件统计
awk '$3>100{high++} $3<=100{low++} END{print "高:"high, "低:"low}' file.txt
```

## 6.3 grep进阶

### 扩展grep

```bash
# egrep或grep -E（扩展正则）
egrep "pattern1|pattern2" file.txt
grep -E "pattern1|pattern2" file.txt

# fgrep或grep -F（固定字符串，不使用正则）
fgrep "literal.string" file.txt
grep -F "literal.string" file.txt
```

### 正则表达式

```bash
# 基本正则
grep "^start" file.txt      # 行首
grep "end$" file.txt         # 行尾
grep "^$" file.txt           # 空行
grep "." file.txt            # 任意字符
grep "a*" file.txt           # 0个或多个a
grep "[abc]" file.txt        # a或b或c
grep "[^abc]" file.txt       # 非a、b、c
grep "[a-z]" file.txt        # 小写字母
grep "[0-9]" file.txt        # 数字

# 扩展正则（需要-E）
grep -E "a+" file.txt        # 1个或多个a
grep -E "a?" file.txt        # 0个或1个a
grep -E "a{3}" file.txt      # 恰好3个a
grep -E "a{2,5}" file.txt    # 2到5个a
grep -E "a{2,}" file.txt     # 至少2个a
grep -E "(ab)+" file.txt     # 分组
grep -E "a|b" file.txt       # a或b

# 特殊字符类
grep -E "\b word\b" file.txt # 单词边界
grep -E "\d+" file.txt       # 数字（某些版本）
grep -P "\d+" file.txt       # Perl正则
```

### 高级用法

```bash
# 显示匹配的颜色
grep --color=auto "pattern" file.txt

# 只显示匹配的部分
grep -o "pattern" file.txt

# 统计匹配次数
grep -c "pattern" file.txt

# 显示不匹配的行
grep -v "pattern" file.txt

# 多个文件
grep "pattern" file1 file2 file3

# 递归搜索
grep -r "pattern" /path
grep -R "pattern" /path  # 跟随符号链接

# 只搜索特定类型文件
grep -r --include="*.txt" "pattern" /path
grep -r --exclude="*.log" "pattern" /path
grep -r --exclude-dir="node_modules" "pattern" /path

# 显示文件名
grep -H "pattern" file.txt   # 显示文件名
grep -h "pattern" file.txt   # 不显示文件名

# 限制匹配数量
grep -m 5 "pattern" file.txt  # 最多5个匹配

# 静默模式（只返回状态码）
grep -q "pattern" file.txt && echo "找到了"
```

## 6.4 cut - 列提取工具

```bash
# 提取第1列（默认制表符分隔）
cut -f1 file.txt

# 提取多列
cut -f1,3 file.txt
cut -f1-3 file.txt      # 第1到第3列
cut -f1,3-5 file.txt    # 第1列和第3到第5列

# 指定分隔符
cut -d: -f1 /etc/passwd
cut -d, -f1,3 data.csv

# 按字符位置提取
cut -c1-5 file.txt      # 每行的第1到第5个字符
cut -c1,3,5 file.txt    # 第1、3、5个字符
cut -c5- file.txt       # 从第5个字符到行尾

# 补充分隔符到输出
cut -d: -f1,3 --output-delimiter='|' /etc/passwd

# 示例：提取用户名和shell
cut -d: -f1,7 /etc/passwd

# 提取IP地址
ifconfig | grep "inet " | cut -d: -f2 | cut -d' ' -f1
```

## 6.5 paste - 合并文件

```bash
# 并排合并文件
paste file1.txt file2.txt

# 指定分隔符
paste -d: file1.txt file2.txt
paste -d',' file1.txt file2.txt

# 合并多个文件
paste file1 file2 file3

# 将文件转为单行（用分隔符连接）
paste -s file.txt
paste -sd',' file.txt    # 用逗号连接

# 示例：创建配置文件
paste -d= keys.txt values.txt > config.ini
```

## 6.6 tr - 字符转换

```bash
# 小写转大写
echo "hello" | tr 'a-z' 'A-Z'
tr 'a-z' 'A-Z' < file.txt

# 大写转小写
tr 'A-Z' 'a-z' < file.txt

# 删除字符
echo "hello123" | tr -d '0-9'    # 删除数字
echo "hello" | tr -d 'l'         # 删除l

# 压缩重复字符
echo "hello    world" | tr -s ' '     # 压缩空格
echo "hellooo" | tr -s 'o'            # 压缩o

# 字符替换
echo "hello" | tr 'aeiou' '12345'    # a->1, e->2...

# 删除非数字字符
echo "abc123def" | tr -cd '0-9'

# 删除换行符
tr -d '\n' < file.txt

# 将空格替换为换行
echo "a b c d" | tr ' ' '\n'

# ROT13加密
echo "hello" | tr 'A-Za-z' 'N-ZA-Mn-za-m'

# 删除Windows换行符
tr -d '\r' < windows.txt > unix.txt
```

## 6.7 sort高级用法

```bash
# 数字排序
sort -n file.txt
sort -n -r file.txt     # 逆序

# 人类可读大小排序
sort -h file.txt

# 随机排序
sort -R file.txt

# 按月份排序
sort -M file.txt

# 按列排序
sort -k2 file.txt           # 按第2列
sort -k2,2 file.txt         # 只按第2列
sort -k2n file.txt          # 第2列数字排序
sort -k1,1 -k2n file.txt    # 第1列字母，第2列数字

# 指定分隔符
sort -t: -k3n /etc/passwd   # 按UID排序

# 去重并排序
sort -u file.txt

# 检查是否已排序
sort -c file.txt

# 合并已排序文件
sort -m sorted1.txt sorted2.txt

# 只输出重复行
sort file.txt | uniq -d

# 示例：按文件大小排序
ls -lh | tail -n +2 | sort -k5h

# 按IP地址排序
sort -t. -k1,1n -k2,2n -k3,3n -k4,4n ips.txt
```

## 6.8 uniq高级用法

```bash
# 基本去重（需先排序）
sort file.txt | uniq

# 统计重复次数
sort file.txt | uniq -c

# 只显示重复行
sort file.txt | uniq -d

# 只显示不重复的行
sort file.txt | uniq -u

# 忽略大小写
sort file.txt | uniq -i

# 只比较前N个字符
uniq -w 10 file.txt

# 跳过前N个字段
uniq -f 2 file.txt

# 示例：查找重复的IP
awk '{print $1}' access.log | sort | uniq -c | sort -rn

# 统计访问最多的URL
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -10
```

## 6.9 文本比较工具

### diff - 比较文件差异

```bash
# 基本比较
diff file1.txt file2.txt

# 并排显示
diff -y file1.txt file2.txt

# 统一格式（常用于补丁）
diff -u file1.txt file2.txt

# 上下文格式
diff -c file1.txt file2.txt

# 递归比较目录
diff -r dir1 dir2

# 忽略空白
diff -w file1.txt file2.txt

# 忽略大小写
diff -i file1.txt file2.txt

# 只显示是否不同
diff -q file1.txt file2.txt

# 生成补丁
diff -u original.txt modified.txt > changes.patch

# 应用补丁
patch original.txt < changes.patch
```

### comm - 比较已排序文件

```bash
# 显示三列：只在file1、只在file2、共有
comm file1.txt file2.txt

# 只显示file1独有的行
comm -23 file1.txt file2.txt

# 只显示file2独有的行
comm -13 file1.txt file2.txt

# 只显示共有的行
comm -12 file1.txt file2.txt
```

## 6.10 列式数据处理

### column - 列格式化

```bash
# 自动列对齐
column -t file.txt

# 指定分隔符
column -t -s: /etc/passwd

# CSV转表格
column -t -s, data.csv

# 示例：美化mount输出
mount | column -t
```

### join - 按键连接文件

```bash
# 连接两个文件（按第1列）
join file1.txt file2.txt

# 指定连接字段
join -1 2 -2 1 file1.txt file2.txt  # file1的第2列与file2的第1列

# 指定分隔符
join -t: file1.txt file2.txt

# 显示不匹配的行
join -a1 file1.txt file2.txt  # file1的不匹配行
join -a2 file1.txt file2.txt  # file2的不匹配行
join -a1 -a2 file1.txt file2.txt  # 所有不匹配行
```

## 6.11 实用脚本示例

### 示例1：日志分析

```bash
#!/bin/bash
# 分析Apache访问日志

LOG="/var/log/apache2/access.log"

echo "=== 访问统计 ==="
echo "总访问次数: $(wc -l < $LOG)"
echo ""

echo "=== Top 10 IP ==="
awk '{print $1}' $LOG | sort | uniq -c | sort -rn | head -10
echo ""

echo "=== Top 10 URL ==="
awk '{print $7}' $LOG | sort | uniq -c | sort -rn | head -10
echo ""

echo "=== 状态码分布 ==="
awk '{print $9}' $LOG | sort | uniq -c | sort -rn
echo ""

echo "=== 每小时访问量 ==="
awk '{print $4}' $LOG | cut -d: -f2 | sort | uniq -c
```

### 示例2：CSV数据处理

```bash
#!/bin/bash
# 处理CSV数据

CSV_FILE="data.csv"

# 提取特定列
echo "=== 提取第2和第4列 ==="
awk -F, '{print $2, $4}' $CSV_FILE

# 计算平均值
echo "=== 第3列平均值 ==="
awk -F, '{sum+=$3; count++} END {print sum/count}' $CSV_FILE

# 过滤数据
echo "=== 第3列大于100的行 ==="
awk -F, '$3 > 100' $CSV_FILE

# 排序
echo "=== 按第3列排序 ==="
sort -t, -k3n $CSV_FILE
```

### 示例3：文本清洗

```bash
#!/bin/bash
# 清洗文本文件

INPUT="raw_data.txt"
OUTPUT="cleaned_data.txt"

# 删除空行、注释、前后空格
sed '/^$/d; /^#/d; s/^[ \t]*//; s/[ \t]*$//' $INPUT > $OUTPUT

echo "清洗完成：$OUTPUT"
```

### 示例4：批量文件重命名

```bash
#!/bin/bash
# 批量重命名文件（小写转大写）

for file in *.txt; do
    newname=$(echo $file | tr 'a-z' 'A-Z')
    mv "$file" "$newname"
    echo "重命名: $file -> $newname"
done
```

### 示例5：生成报告

```bash
#!/bin/bash
# 系统报告生成器

REPORT="system_report.txt"

{
    echo "=== 系统报告 ==="
    echo "生成时间: $(date)"
    echo ""

    echo "=== 磁盘使用 ==="
    df -h | column -t
    echo ""

    echo "=== 内存使用 ==="
    free -h | column -t
    echo ""

    echo "=== Top 10 进程 ==="
    ps aux | sort -rn -k3 | head -11 | column -t
    echo ""

    echo "=== 登录用户 ==="
    who | column -t

} > $REPORT

echo "报告已生成: $REPORT"
```

## 6.12 性能优化技巧

### 选择合适的工具

```bash
# 简单替换 -> sed
sed 's/old/new/g' file.txt

# 列处理 -> awk
awk '{print $1, $3}' file.txt

# 搜索 -> grep
grep "pattern" file.txt

# 简单提取 -> cut
cut -d: -f1 /etc/passwd
```

### 管道优化

```bash
# 不好：多次读取文件
cat file.txt | grep pattern | sort | uniq

# 好：减少管道
grep pattern file.txt | sort -u

# 不好：不必要的cat
cat file.txt | awk '{print $1}'

# 好：直接处理
awk '{print $1}' file.txt
```

### 大文件处理

```bash
# 使用流处理，不要全部读入内存
sed 's/old/new/g' huge_file.txt > output.txt

# 处理部分数据
head -n 10000 huge_file.txt | awk '{print $1}'

# 并行处理
cat huge_file.txt | parallel --pipe grep pattern
```

## 6.13 正则表达式速查

### 基本元字符

```
.       # 任意单个字符
^       # 行首
$       # 行尾
*       # 0次或多次
+       # 1次或多次（扩展）
?       # 0次或1次（扩展）
[]      # 字符集
[^]     # 否定字符集
\       # 转义字符
|       # 或（扩展）
()      # 分组（扩展）
```

### 字符类

```
[a-z]       # 小写字母
[A-Z]       # 大写字母
[0-9]       # 数字
[a-zA-Z]    # 所有字母
[^0-9]      # 非数字
[:digit:]   # 数字
[:alpha:]   # 字母
[:alnum:]   # 字母和数字
[:space:]   # 空白字符
```

### 量词

```
{n}         # 恰好n次
{n,}        # 至少n次
{n,m}       # n到m次
*           # {0,}
+           # {1,}
?           # {0,1}
```

### 实用模式

```bash
# IP地址
grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' file.txt

# 邮箱
grep -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' file.txt

# URL
grep -E 'https?://[a-zA-Z0-9./?=_-]+' file.txt

# 日期 (YYYY-MM-DD)
grep -E '[0-9]{4}-[0-9]{2}-[0-9]{2}' file.txt

# 时间 (HH:MM:SS)
grep -E '[0-9]{2}:[0-9]{2}:[0-9]{2}' file.txt

# MAC地址
grep -E '([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}' file.txt
```

## 6.14 实践练习

### 练习1：sed替换

```bash
# 创建测试文件
cat > test.txt << EOF
Hello World
Hello Linux
Hello Ubuntu
EOF

# 1. 将所有Hello替换为Hi
# 2. 删除包含Linux的行
# 3. 在每行开头添加"-> "
```

### 练习2：awk数据处理

```bash
# 创建成绩文件
cat > scores.txt << EOF
张三 85 90 88
李四 92 88 95
王五 78 85 82
EOF

# 1. 计算每个学生的平均分
# 2. 找出平均分最高的学生
# 3. 统计总平均分
```

### 练习3：日志分析

```bash
# 模拟访问日志
cat > access.log << EOF
192.168.1.1 - [2026-01-07] GET /index.html 200
192.168.1.2 - [2026-01-07] GET /about.html 200
192.168.1.1 - [2026-01-07] GET /contact.html 404
192.168.1.3 - [2026-01-07] POST /api/login 200
EOF

# 1. 统计每个IP的访问次数
# 2. 统计每种状态码的数量
# 3. 提取所有404错误的URL
```

## 下一步

完成本章后，你应该：
- ✅ 掌握sed的基本和高级用法
- ✅ 熟练使用awk处理列式数据
- ✅ 掌握grep的正则表达式
- ✅ 了解其他文本处理工具
- ✅ 能够编写文本处理脚本

**准备好了吗？让我们进入[第7章：软件包管理](../07-软件包管理/README.md)！**

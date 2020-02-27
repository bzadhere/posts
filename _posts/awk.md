---
title: awk
date: 2020-01-09 14:07:54
tags:
cateogries: linux
---



# 调用方法

```shell
// -F 每次读取一行，未设置-F默认用空格为域分隔符
awk [-F field-separator] 'commands' input-file(s)

awk -f awk-script-file input-file(s)

awk-script-file 直接执行
```



# AWK模式和操作

```shell
awk 'BEGIN{ commands } pattern{ commands } END{ commands }' file
```

*  第一步：执行`BEGIN{commands}`语句块中的语句，可选的语句块，比如变量初始化、打印输出表格的表头等语句。
*  第二步：**循环**从文件或标准输入(stdin)读取一行，然后执行`pattern{ commands }`语句块;  通用命令是最重要的部分，它也是可选的。如果没有提供pattern语句块，则默认执行`{ print }`，即打印每一个读取到的行，awk读取的每一行都会执行该语句块。
*  第三步：从输入流中读取完所有的行**之后**即被执行 `END{ commands }`。 脚本通常是被**单引号**或**双引号**中 。

```shell
// 统计文件行数
awk 'BEGIN{ i=0 } { i++ } END{ print i }' file
awk "BEGIN{ i=0 } { i++ } END{ print i }" file

```

# awk内置变量

 说明：\[A\]\[N\]\[P\]\[G\]表示第一个支持变量的工具，[A]=awk、[N]=nawk、[P]=POSIXawk、[G]=gawk 

```shell
$n 当前记录的第n个字段，比如n为1表示第一个字段，n为2表示第二个字段。 
$0 这个变量包含执行过程中当前行的文本内容。
[N] ARGC 命令行参数的数目。
[G] ARGIND 命令行中当前文件的位置（从0开始算）。
[N] ARGV 包含命令行参数的数组。
[G] CONVFMT 数字转换格式（默认值为%.6g）。
[P] ENVIRON 环境变量关联数组。
[N] ERRNO 最后一个系统错误的描述。
[G] FIELDWIDTHS 字段宽度列表（用空格键分隔）。
[A] FILENAME 当前输入文件的名。
[P] FNR 同NR，但相对于当前文件。
[A] FS 字段分隔符（默认是任何空格）。
[G] IGNORECASE 如果为真，则进行忽略大小写的匹配。
[A] NF 表示字段数，在执行过程中对应于当前的字段数。
[A] NR 表示记录数，在执行过程中对应于当前的行号。
[A] OFMT 数字的输出格式（默认值是%.6g）。
[A] OFS 输出字段分隔符（默认值是一个空格）。
[A] ORS 输出记录分隔符（默认值是一个换行符）。
[A] RS 记录分隔符（默认是一个换行符）。
[N] RSTART 由match函数所匹配的字符串的第一个位置。
[N] RLENGTH 由match函数所匹配的字符串的长度。
[N] SUBSEP 数组下标分隔符（默认值是34）。
```

```shell
// 统计文件行号
awk 'END{ print NR }' file 

// 执行linux的date命令，并通过管道输出给getline，然后再把输出赋值给自定义变量out，并打印它
awk 'BEGIN{ "date" | getline out; print out }' 
```



# 流程控制语句

 每条命令语句后面可以用`;`**分号**结尾。 

```shell
if(表达式)
  {语句1}
else if(表达式)
  {语句2}
else
  {语句3}

while(表达式)
  {语句}
  
for(变量 in 数组)
  {语句}
  
for(变量;条件;表达式)
  {语句}

do
{语句} while(条件)

break 当 break 语句用于 while 或 for 语句时，导致退出程序循环。
continue 当 continue 语句用于 while 或 for 语句时，使程序循环移动到下一个迭代。
next 能能够导致读入下一个输入行，并返回到脚本的顶部。这可以避免对当前输入行执行其他的操作过程。
exit 语句使主输入循环退出并将控制转移到END,如果END存在的话。如果没有定义END规则，或在END中应用exit语句，则终止脚本的执行。
```

```shell
awk 'BEGIN{
test=100;
total=0;
while(i<=test){
  total+=i;
  i++;
}
print total;
}'
```



# 内置函数



# 一般函数


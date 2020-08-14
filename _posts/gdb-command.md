---
title: gdb  command
date: 2019-11-07 11:28:24
tags:
categories: gdb
---

[参考地址](https://github.com/hellogcc/100-gdb-tips)

# 脚本
用户目录下 .gdbinit 文件, 启动gdb时默认加载

<!-- more -->

```
# 打印STL容器中的内容
python
import sys
sys.path.insert(0, '/home/maude/gdb_printers/python')
from libstdcxx.v6.printers import register_libstdcxx_printers
register_libstdcxx_printers (None)
end

# 保存历史命令
set history filename ~/.gdb_history
set history save on

# 退出时不显示提示信息
set confirm off

# 按照派生类型打印对象
set print object on

# 打印数组的索引下标
set print array-indexes on

# 每行打印一个结构体成员
set print pretty on
```

# 源文件
directory（或dir)命令设置源文件的查找目录

# 断点

```
break/b
break file:line
break calss:function

// 设置条件断点
(gdb) break do_freebie if a> 0 && b>0

// 调试汇编, 在程序地址上打印断点, b *address 

// 设置临时断点, 只生效一次
(gdb) tbreak filename:line

// 使用如下命令将设置的断点保存下来
(gdb) save breakpoints file-name-to-save
// 下次调试时, 加载保存的断点
(gdb) source file-name-to-save

```

# 函数

```
// 从断点开始继续执行
continue/c  

// 不进入的单步执行
next/n

// 进入的单步执行
step/s 

// nexti
Usage: nexti [N]
Argument N means step N times (or till program stops for another reason).

// jump 
Usage: jump <location>
Give as argument either LINENUM or *ADDR, where ADDR is an expression

// 执行完循环体, 后面的一个语句停下
until

// 退出正在调试的函数
(gdb) finish

// 选择函数栈帧
(gdb) frame 2
(gdb) i frame

// 向上或向下移动栈帧
(gdb) up n
(gdb) down n
```

# 观察点

```
// 当var值变化时, 程序会停下来
(gdb) watch var
(gdb) info watchpoints

// 设置观察点只针对特定线程
// 当线程1改变var的值时程序会停下来, 其他线程改变不会停止
(gdb) watch var thread 1

// 读观察点, 当发生读取变量行为时，程序就会暂停住
(gdb) rwatch var

// 当发生读取变量或改变变量值的行为时，程序就会暂停住
(gdb) awatch var
```

# catchepoint

```
// 设置catchpoint只触发一次
(gdb) tcatch fork

// 当fork调用发生后, gdb会暂停程序的运行
(gdb) catch fork

// 为系统调用设置catchpoint
(gdb) catch syscall mmap

(gdb) help catch
Set catchpoints to catch events.

List of catch subcommands:

catch assert -- Catch failed Ada assertions
catch catch -- Catch an exception
catch exception -- Catch Ada exceptions
catch exec -- Catch calls to exec
catch fork -- Catch calls to fork
catch load -- Catch loads of shared libraries
catch rethrow -- Catch an exception
catch signal -- Catch signals by their names and/or numbers
catch syscall -- Catch system calls by their names and/or numbers
catch throw -- Catch an exception
catch unload -- Catch unloads of shared libraries
catch vfork -- Catch calls to vfork

```

# 打印

```
(gdb) help x
Examine memory: x/FMT ADDRESS.
ADDRESS is an expression for the memory address to examine.
FMT is a repeat count followed by a format letter and a size letter.
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal),
  t(binary), f(float), a(address), i(instruction), c(char), s(string)
  and z(hex, zero padded on the left).
Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).
The specified number of objects of the specified size are printed
according to the format.

Defaults for format and size letters are those previously used.
Default count is 1.  Default address is following last thing printed

// 打印字符串
(gdb) x/s str1

// 格式为“x/nfu addr“, n 是输出单元个数, f 是格式, u 是单元的长度 
// 以16进制格式打印数组前32个byte的值
(gdb) x/32xb array

// 打印STL容器, 加载python脚本后增加可读性

// 打印大数组
(gdb) set print elements unlimited
(gdb) p array

// 打印数组中任意连续元素值
(gdb) p array[60]@10

// 打印函数局部变量的值, bt full [N] 默认打印全部栈函数中的变量, N是栈个数
(gdb) bt full
#0  fun_a () at a.c:6
        a = 0
#1  0x000109b0 in fun_b () at a.c:12
        b = 1
#2  0x000109e4 in fun_c () at a.c:19
        c = 2
		

// 只打印当前函数局部变量的值
(gdb) info locals

// 查看进程的内存映射信息
(gdb)info proc mappings

// 查看进程的内存信息，包括引用的动态链接库等
(gdb)info files
(gdb)info target

// 打印静态变量, 可以显式地指定文件名
(gdb) p 'static-1.c'::var
$1 = 1

// 打印栈中的变量, 对于C++的函数名，需要使用单引号括起来
(gdb) bt
(gdb) p func2::b
(gdb) p '::namespace::Student'::n->pi->dump()


// 打印变量类型和变量所在文件
(gdb) whatis var
type = struct child

(gdb) ptype var
type = struct child {
    char name[10];
    enum {boy, girl} gender;
}

(gdb) info variables var

// 打印源代码行
(gdb) list [N,M]

// 每行只会显示结构体的一名成员
(gdb) set print pretty on
(gdb) p st

// 打开一个新的终端, 指定输入输出设备(printf打印输出到这个终端)
51_zjdev[/data01/zjgrp/zjdev]%tty
/dev/pts/2

(gdb) tty /dev/pts/2
(gdb) r

$ gdb -tty /dev/pts/2 ./a.out
(gdb) r

```

# 多进程/线程

```
// 查看多少个进程
(gdb) info inferiors
  Num  Description       Executable        
* 1    process 6070      /data01/zjgrp/zjdev/ob_rel/bin/sframe.2.3.0.907313 

// 只允许一个线程运行, 忘记的话help set
(gdb) set scheduler-locking on

// $_thread 打印线程号

// $_exitcode 记录程序退出时的“exit code”
```


# 其他

## 启动参数
```
// 启动前设置参数
$ gdb -args ./a.out a b c

// 启动前设置参数
(gdb) set args a b c
(gdb) show args

// 运行时指定
(gdb) r a b

```

## 记录调试过程
```
// 默认的日志文件是“gdb.txt”
(gdb) set logging file log.txt
// 把执行gdb的过程记录下来
(gdb) set logging on
```

## 缩写
```
b -> break
c -> continue
d -> delete
f -> frame
i -> info
j -> jump
l -> list
n -> next
p -> print
r -> run
s -> step
u -> until

aw -> awatch
bt -> backtrace
dir -> directory
disas -> disassemble
fin -> finish
ig -> ignore
ni -> nexti
rw -> rwatch
si -> stepi
tb -> tbreak
wa -> watch
win -> winheight
```














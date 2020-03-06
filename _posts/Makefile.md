---
title: Makefile
date: 2018-12-19 14:31:02
tags: Makefile
categories: Makefile
---

Makefile 包含 变量/注释/文件指示/显式规则/隐晦规则

# 变量
= 基本赋值, 可以使用后面的变量
:= 覆盖之前的值, 前面的变量不能使用后面的变量
?= 如果没被赋值过就赋值
+= 追加赋值，用空格隔开方式追加
$(shell cmd) 用shell函数赋值
\$(var:<parten>=<parten>) 替换
环境变量可以直接使用
嵌套调用Makefile, export 声明变量传递给下一层Makefile

```makefile
# 赋值
foo = $(bar)
bar = a.cpp b.cpp
# 嵌套赋值会报错
A = $(B)
B = $(A)
# := 赋值, 只是不能使用后面的变量, foo是a.cpp
foo := $(bar) a.cpp
bar := b.cpp
# ?= 如果foo未被定义过则赋值, 否则什么也不做
foo ?= bar 
# += 追加
foo = a.cpp
foo += b.cpp
# shell赋值
dir = $(shell pwd)
# 替换 将字符串结尾的a替换成b, 结尾指空格和结束符
$(var:a=b)
$(var:%.o=%.c)
# @echo 打印变量, 没有@时会回显命令
@echo "bar=>$(bar)"

```

__目标变量__
foo.o : foo.cpp
$(CXX) $(CFLAG) foo.cpp

# 注释
```
# 注释
bar = a.cpp
```

# 文件指指示
```makefile
# 包含NGCmake
include NGCmake
```



# 函数
```makefile
$(function <arg>) 或 ${function <arg>}
```



## 字符串操作

|函数|名称|功能|返回|
|:-|:-|:-|:-|
|$(subst &lt;from&gt;,&lt;to&gt;,&lt;text&gt;)| 字符串替换|把text里from替换成to | 替换后的字符串|
|$(patsubst &lt;pattern&gt;,&lt;replacement&gt;,&lt;text&gt;) | 模式字符串替换|text中以空格、回车、TAB是否符合pattern, 若符合, 用replacement替换| 替换后的字符串|
|$(strip &lt;string&gt;)| 去空格| 去掉开头和结尾空字符| 处理后的字符串|
|$(findstring &lt;find&gt;,&lt;in&gt;)| 查找字符串|在in中查找find | 找到返回find，否则返回空|
|$(filter &lt;pattern...&gt;,&lt;text&gt;) | 过滤函数|保留符合pattern的单词，可以有多个模式| 符合的字符串 |
|$(filter-out &lt;pattern...&gt;,&lt;text&gt;)| 去除函数| 去除符合模式的单词| 处理后的字符串|
|$(srot &lt;list&gt;)| 排序| 单词按升序排 | 排序后字符串|
|$(word &lt;n&gt;,&lt;text&gt;)|取单词| 取第n个单词, 从1开始| 单词或空|
|$(wordlist &lt;n&gt;,&lt;m&gt;,&lt;text&gt;) | 取单词| 去n到m的单词| 单词或空|
|$(words &lt;text&gt;)| 统计个数| 统计单词个数| 单词数量|
|$(firstword &lt;text&gt;)| 取首个单词| 等价于$(word 1,&lt;text&gt;)| 首个单词|

```
CFLAGS+=$(patsubst %,-I%,$(subst :, ,$(VPATH)))
```

## 文件名操作 
|函数|名称|功能|返回|
|:-|:-|:-|:-|
|$(dir &lt;names...&gt;) | 取目录|取出names反斜杠的目录| 目录或./|
|$(notdir &lt;names...&gt;)| 取文件名| 去掉反斜杠前面的目录部分| 文件名序列|
|$(suffix &lt;names...&gt;)| 取文件序列后缀| 去掉非后缀部分| .cpp等后缀 或 空|
|$(basename &lt;names...&gt;)| 取文件序列前缀| 去掉后缀部分| |
|$(addsuffix &lt;suffix&gt;,&lt;names...&gt;)| 加后缀| 加后缀| 文件序列名|
|$(addprefix &lt;prefix&gt;,&lt;names...&gt;)| 加前缀| 加前缀| 文件序列名|
|$(join &lt;list1&gt;,&lt;list2&gt;)| 链接|list1+list2 | 链接后的字符串|


## 控制和调用
+ $(foreach &lt;var&gt;,&lt;list&gt;,&lt;text&gt;)
循环执行

```Makefile
names := a b c d
files := $(foreach n,$(names),$(n).o)
# ouput: a.o b.o c.o d.o
```

+ $(if &lt;condition&gt;,&lt;then-part&gt;)
```makefile
ifeq ($(CXX), gcc)
$(CXX) -o test $(object) $(lib_gcc)
else
$(CXX) -o test $(object) $(lib_others)
endif
```

+ $(call &lt;expression&gt;,&lt;param1&gt;,&lt;param2&gt;...)
创建新的函数
```
reserve := $(1) $(2)
f = $(call reserve,a,b)

f的值是a b 
```

+ $(origin &lt;var&gt;)
变量是哪里来的, undefined, default, file, command line, override, automatic

## shell函数
text := $(shell cat foo)

## 控制Make函数
|函数|名称|功能|返回|
|:-|:-|:-|:-|
|$(error &lt;text...&gt;) | 产生一个致命错误|提示错误信息| 提示|
|$(warning &lt;text...&gt;) | 输出告警信息 |提示告警信息| 提示|

# 显示规则
```Makefile
# 通配符: */%/[...]
targest : prerequisites
	command
```

## 文件搜索
// 为符合模式的文件指定搜索目录
vpath <pattern> <directories>
// 为符合模式的文件清除搜索目录
vpath <pattern>
// 清楚搜索目录
vpath

## 伪目标

```makefile
# make clean会执行 "rm -rf *.o"
# 但如果当前目录下存在clean文件, 规则没有依赖文件，目标被认为是最新的，不会去执行rm命令
clean:
	rm -rf *.o 

# .PHONY 显示的标记一个伪目标, 还能避免重名
# 工作目录下存在clean文件也会执行rm命令
.PHONY: clean
clean:
	rm -rf *.o 
	
# 多个伪目标，伪目标之间有依赖
# make cleanobj 相当于一个子程序
.PHONY: cleanall cleanobj cleandiff 
cleanall : cleanobj cleandiff 
	rm program 
cleanobj : 
	rm *.o 
cleandiff : 
	rm *.diff 
	
# 多个伪目标，每个伪目标有自己的依赖
# make all, 重建all依赖的文件(prog1,prog2,prog3)
all: prog1 prog2 prog3
.PHONY: all
.PHONY : all 
prog1 : prog1.o utils.o 
	cc -o prog1 prog1.o utils.o 
prog2 : prog2.o 
	cc -o prog2 prog2.o 
prog3 : prog3.o 
	cc -o prog3 prog3.o

```

## 静态规则
<targets...> : <target-pattern> : <prerequisites>
<commands>
...

```
object = foo.o bar.o 
all: $(object)
$(object): %.o: %.c 
$CC -c $(FLAG) $< -o $@
```

## 自动生成依赖关系
cc -M name.c 会生成name.d 依赖关系文件

# 隐含规则
## 自动推到
|语言| 规则 | 命令|
|:-|:-|:-|
|C|n.o 目标文件自动退到依赖n.c| $(CC) -c $(CPPFLAGS) $(CFLAGS)|
|C++|n.o 目标文件自动退到依赖n.cc| $(CXX) -c $(CPPFLAGS) $(CFLAGS)|

## 模式规则
规则中包含%, 自动化变量在模式规则中使用
```
%.o : %.c
$(CC) -c $(CFLAGS) $(CPPFLAGS) $&lt; -o $@
```

## 自动化变量
|变量| 含义|
|:-|:-|
|$@| 目标文件集, 在模式规则中表示符合模式定义的集合|
|$%| 目标是库文件(.a/.lib), 则表示目标成员; 否则为空|
|$&lt;| 依赖列表的第一个文件名称, 依赖目标是以模式(%)定义,则表示符合模式的文件集|
|$?| 时间戳比目标文件新的依赖文件列表, 空格分隔|
|$^| 依赖目标集合, 同一个文件不可重复|
|$+| 依赖目标集合, 同一个文件可重复|
|$*| 目标模式中"%"及其之前部分|

```
foo.a : bar.o 
$% 表示 bar.o, $@ 表示 foo.a
```

## 重载内建隐含规则


# 实践分析














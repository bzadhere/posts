---
title: CPU profiler
date: 2019-03-21 15:00:52
tags:
categories: tools
---

使用CPU profiler有三部分: 库连接, 运行代码, 分析输出。
CPU profiler的数据文件有单独的说明文档...

# 库连接
1. 编译时连接 -lprofiler
2. 设置全局环境变量, setenv LD_PRELOAD /usr/lib/libprofiler.so(不推荐, 用户设置没有限制)

以上两种方式并没有打开CPU profiler,仅仅只是在代码中插入链接符号.

```
g++ test.cpp -o test -lprofiler
```
<!-- more -->
# 运行代码
有三种方式可以打开CPU profiler:
1. 设置CPUPROFILE环境变量, 例如setenv CPUPROFILE ~/data/prof/decode.prof

2. 设置CPUPROFILESIGNAL, 定义未使用的信号控制输出
```
setenv CPUPROFILE ~/data/prof/test.prof
setenv CPUPROFILESIGNAL 60
// 开始
kill -60 pid
// 停止
kill -60 pid
```

3. include 头文件<gperftools/profiler.h>, 使用ProfilerStart(char* filename) 和 ProfilerStop()
还有ProfilerFlush() and ProfilerStartWithOptions()

出于安全考虑, CPU分析不会写到文件中, 不能应用于setuid程序。

## 环境变量
| name | desc |
|:-|:-|
| CPUPROFILE_FREQUENCY | 默认100samples/second, 相当于一个样本10ms|
| CPUPROFILE_REALTIME| 默认不设置, 只要设置了(即使是0或空), 用ITIMER_REAL代替 ITIMER_PROF, 一般不设置|


# 分析输出
pprof是用于分析profile的脚本。有多种输出模式, 有类似gcc -pg 的输出(gprof)。
pprof依赖dot/gv。

```
pprof <format> [options] [binary] <source>


% pprof /bin/ls ls.prof
                       Enters "interactive" mode
% pprof --text /bin/ls ls.prof
                       Outputs one line per procedure
% pprof --gv /bin/ls ls.prof
                       Displays annotated call-graph via 'gv'
% pprof --gv --focus=Mutex /bin/ls ls.prof
                       Restricts to code paths including a .*Mutex.* entry
% pprof --gv --focus=Mutex --ignore=string /bin/ls ls.prof
                       Code paths including Mutex but not string
% pprof --list=getdir /bin/ls ls.prof
                       (Per-line) annotated source listing for getdir()
% pprof --disasm=getdir /bin/ls ls.prof
                       (Per-PC) annotated disassembly for getdir()
% pprof --text localhost:1234
                       Outputs one line per procedure for localhost:1234
% pprof --callgrind /bin/ls ls.prof
                       Outputs the call information in callgrind format
```

## 分析text输出
```    
eg.
  14   2.1%  17.2%       58   8.7% std::_Rb_tree::find

```

| value| desc |
|:-|:-|
| 14 | samples |
| 2.1% | 在总samples的比例 |
| 17.2% | 到该函数为止, 已经运行的函数占总samples的比例 |
| 58 | 函数加上函数里被调用者的总samples |
| 8.7% | 函数加上函数里被调用者的总samples 在总samples比例 |
| std::_Rb_tree::find | 函数名 |

## 分析Callgrind输出
```
% pprof --callgrind /bin/ls ls.prof > ls.callgrind
% kcachegrind ls.callgrind

```


## 



# 网址

 https://github.com/google/pprof 

 https://zhuanlan.zhihu.com/p/59437135 
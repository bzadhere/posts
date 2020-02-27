---
title: strace命令
date: 2019-01-14 15:24:54
tags:
categories: linux
---

# 一个简单的例子
```
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

int main()
{
  int fd;
  int i = 0;
  fd = open("./test.cpp", O_RDONLY);
  if(fd < 5)
    i = 5;
  else
    i = 2;
  return i;
}

g++ test.cpp -o test
strace -o test.strace ./test

vim test.strace
 1 execve("./test", ["./test"], [/* 32 vars */]) = 0
 2 brk(NULL)                               = 0x149b000
 3 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fb83d939000
 4 access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
 5 open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
 6 fstat(3, {st_mode=S_IFREG|0644, st_size=54597, ...}) = 0
 7 mmap(NULL, 54597, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fb83d92b000
 8 close(3)                                = 0
 9 open("/usr/lib64/libstdc++.so.6", O_RDONLY|O_CLOEXEC) = 3
10 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0 \262\5\0\0\0\0\0"..., 832) = 832
11 fstat(3, {st_mode=S_IFREG|0755, st_size=991616, ...}) = 0
12 mmap(NULL, 3171168, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fb83d412000
13 mprotect(0x7fb83d4fb000, 2093056, PROT_NONE) = 0
14 mmap(0x7fb83d6fa000, 40960, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0xe8000) = 0x7fb83d        6fa000
15 mmap(0x7fb83d704000, 82784, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fb83d70400        0
16 close(3)                                = 0
17 open("/usr/lib64/libm.so.6", O_RDONLY|O_CLOEXEC) = 3
18 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\20S\0\0\0\0\0\0"..., 832) = 832
19 fstat(3, {st_mode=S_IFREG|0755, st_size=1137016, ...}) = 0
20 mmap(NULL, 3150120, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fb83d110000 
21 mprotect(0x7fb83d211000, 2093056, PROT_NONE) = 0
22 mmap(0x7fb83d410000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x100000) = 0x7fb83d        410000
23 close(3)                                = 0
24 open("/usr/lib64/libgcc_s.so.1", O_RDONLY|O_CLOEXEC) = 3
25 read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\220*\0\0\0\0\0\0"..., 832) = 832
26 fstat(3, {st_mode=S_IFREG|0755, st_size=88776, ...}) = 0
27 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fb83d92a000 
28 mmap(NULL, 2184192, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fb83cefa000
29 mprotect(0x7fb83cf0f000, 2093056, PROT_NONE) = 0
30 mmap(0x7fb83d10e000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x14000) = 0x7fb83d1        0e000
31 close(3)                                = 0
32 open("/usr/lib64/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
33 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\340$\2\0\0\0\0\0"..., 832) = 832
34 fstat(3, {st_mode=S_IFREG|0755, st_size=2151672, ...}) = 0
35 mmap(NULL, 3981792, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fb83cb2d000
36 mprotect(0x7fb83ccef000, 2097152, PROT_NONE) = 0
37 mmap(0x7fb83ceef000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1c2000) = 0x7fb83        ceef000
38 mmap(0x7fb83cef5000, 16864, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fb83cef500        0
39 close(3)                                = 0
40 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fb83d929000
41 mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fb83d927000
42 arch_prctl(ARCH_SET_FS, 0x7fb83d927740) = 0
43 mprotect(0x7fb83ceef000, 16384, PROT_READ) = 0
44 mprotect(0x7fb83d10e000, 4096, PROT_READ) = 0
45 mprotect(0x7fb83d410000, 4096, PROT_READ) = 0
46 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fb83d926000
47 mprotect(0x7fb83d6fa000, 32768, PROT_READ) = 0
48 mprotect(0x600000, 4096, PROT_READ)     = 0
49 mprotect(0x7fb83d93a000, 4096, PROT_READ) = 0
50 munmap(0x7fb83d92b000, 54597)           = 0
51 open("./test.cpp", O_RDONLY)            = 3
52 exit_group(5)                           = ?
53 +++ exited with 5 +++

#格式为：系统调用的名称( 参数... ) = 返回值  错误标志和描述
#line1: 对于命令行下执行的程序，execve(或exec系列调用中的某一个)均为strace输出系统调用中的第一个。strace首先调用fork  
或clone函数新建一个子进程，然后在子进程中调用exec载入需要执行的程序(这里为./test)
#line2: 以NULL作为参数调用brk，返回值为内存管理的起始地址(若在子进程中调用malloc，则从 0x149b000 地址开始分配空间)
#line4: 使用mmap函数进行匿名内存映射，以此来获取4096bytes内存空间，该空间起始地址为 0x7fb83d939000
#line3: 调用access函数检验/etc/ld.so.preload是否存在
#line5: 打开/etc/ld.so.cache, 返回文件描述符3
#line6: fstat函数获取/etc/ld.so.cache文件信息
#line7: 调用mmap函数将/etc/ld.so.cache文件映射至内存
#line8: close关闭文件描述符为3指向的/etc/ld.so.cache文件
#line10: 调用read，从/usr/lib64/libstdc++.so.6该libc库文件中读取832bytes，即读取ELF头信息
#line13: 使用 mprotect 函数对 0x7fb83d4fb000 起始的2093056 进行保护(PROT_NONE表示不能访问，PROT_READ表示可以读取)
#line50: 调用 munmap 函数，将/etc/ld.so.cache文件从内存中去映射，与Line 4的mmap对应
#line51: 对应源码中使用到的唯一的系统调用 open 函数，使用其打开./test.cpp文件
#line532: 返回5，退出
```
几乎都用于进行进程初始化工作：装载被执行程序、载入libc函数库、设置内存映射等。
源码中的if语句或其他代码在相应strace输出中并没有体现，因为它们并没有唤起系统调用。  
strace只关心程序与系统之间产生的交互，因而strace不适用于程序逻辑代码的排错和分析。

# 命令选项
> -T:   记录各个系统调用花费的时间，精确到微秒
> -r:   以第一个系统调用(通常为execve)计时，精确到微秒
> -t:   时：分：秒
> -tt:  时：分：秒 . 微秒
> -ttt: 计算机纪元以来的秒数 . 微秒
> -p: 跟踪正在运行的进程

```
[root@zhangbb test]# strace -Tr ./test
     0.000000 execve("./test", ["./test"], [/* 32 vars */]) = 0 <0.000442>
     0.000687 brk(NULL)                 = 0xd49000 <0.000009>
     0.000059 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f57a5783000 <0.000012>
     0.000067 access("/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory) <0.000024>
     0.000098 open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3 <0.000015>
     0.000071 fstat(3, {st_mode=S_IFREG|0644, st_size=54597, ...}) = 0 <0.000010>
     0.000072 mmap(NULL, 54597, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f57a5775000 <0.000060>
     0.000079 close(3)                  = 0 <0.000009>
     0.000044 open("/usr/lib64/libstdc++.so.6", O_RDONLY|O_CLOEXEC) = 3 <0.000036>
     0.000057 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0 \262\5\0\0\0\0\0"..., 832) = 832 <0.000014>
     0.000043 fstat(3, {st_mode=S_IFREG|0755, st_size=991616, ...}) = 0 <0.000010>
     0.000032 mmap(NULL, 3171168, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f57a525c000 <0.000017>
     0.000043 mprotect(0x7f57a5345000, 2093056, PROT_NONE) = 0 <0.000015>
     0.000043 mmap(0x7f57a5544000, 40960, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0xe8000) = 0x7f57a5544000 <0.000014>
     0.000042 mmap(0x7f57a554e000, 82784, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f57a554e000 <0.000012>
     0.000041 close(3)                  = 0 <0.000009>
     0.000041 open("/usr/lib64/libm.so.6", O_RDONLY|O_CLOEXEC) = 3 <0.000015>
     0.000040 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\20S\0\0\0\0\0\0"..., 832) = 832 <0.000011>
     0.000029 fstat(3, {st_mode=S_IFREG|0755, st_size=1137016, ...}) = 0 <0.000010>
     0.000029 mmap(NULL, 3150120, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f57a4f5a000 <0.000012>
     0.000029 mprotect(0x7f57a505b000, 2093056, PROT_NONE) = 0 <0.000013>
     0.000028 mmap(0x7f57a525a000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x100000) = 0x7f57a525a000 <0.000012>
     0.000041 close(3)                  = 0 <0.000009>
     0.000036 open("/usr/lib64/libgcc_s.so.1", O_RDONLY|O_CLOEXEC) = 3 <0.000014>
     0.000037 read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\220*\0\0\0\0\0\0"..., 832) = 832 <0.000010>
     0.000028 fstat(3, {st_mode=S_IFREG|0755, st_size=88776, ...}) = 0 <0.000009>
     0.000037 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f57a5774000 <0.000011>
     0.000032 mmap(NULL, 2184192, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f57a4d44000 <0.000022>
     0.000040 mprotect(0x7f57a4d59000, 2093056, PROT_NONE) = 0 <0.000014>
     0.000030 mmap(0x7f57a4f58000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x14000) = 0x7f57a4f58000 <0.000015>
     0.000049 close(3)                  = 0 <0.000009>
     0.000033 open("/usr/lib64/libc.so.6", O_RDONLY|O_CLOEXEC) = 3 <0.000015>
     0.000033 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\340$\2\0\0\0\0\0"..., 832) = 832 <0.000010>
     0.000028 fstat(3, {st_mode=S_IFREG|0755, st_size=2151672, ...}) = 0 <0.000009>
     0.000029 mmap(NULL, 3981792, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f57a4977000 <0.000012>
     0.000028 mprotect(0x7f57a4b39000, 2097152, PROT_NONE) = 0 <0.000014>
     0.000028 mmap(0x7f57a4d39000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1c2000) = 0x7f57a4d39000 <0.000015>
     0.000045 mmap(0x7f57a4d3f000, 16864, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f57a4d3f000 <0.000012>
     0.000037 close(3)                  = 0 <0.000009>
     0.000050 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f57a5773000 <0.000010>
     0.000039 mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f57a5771000 <0.000014>
     0.000050 arch_prctl(ARCH_SET_FS, 0x7f57a5771740) = 0 <0.000010>
     0.000169 mprotect(0x7f57a4d39000, 16384, PROT_READ) = 0 <0.000014>
     0.000040 mprotect(0x7f57a4f58000, 4096, PROT_READ) = 0 <0.000012>
     0.000054 mprotect(0x7f57a525a000, 4096, PROT_READ) = 0 <0.000013>
     0.000877 mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f57a5770000 <0.000013>
     0.000090 mprotect(0x7f57a5544000, 32768, PROT_READ) = 0 <0.000014>
     0.000033 mprotect(0x600000, 4096, PROT_READ) = 0 <0.000013>
     0.000032 mprotect(0x7f57a5784000, 4096, PROT_READ) = 0 <0.000014>
     0.000029 munmap(0x7f57a5775000, 54597) = 0 <0.000024>
     0.000156 open("./test.cpp", O_RDONLY) = 3 <0.000030>
     0.000097 exit_group(5)             = ?
     0.000147 +++ exited with 5 +++
```






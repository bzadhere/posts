---
title: linux 常用命令
date: 2018-10-24 15:41:49
tags:
categories: linux
---

# 系统级计数器

## 系统信息

```shell
# 查看系统信息，如硬盘
host-10-19-14-51:/sys/block/vda> cat stat 
 7387132  3770038 1127099632 427297720 73091212 35427793 5872626739 1788849144        0 1012238160 2216319780

```





## vmstat

查看整个机器的CPU,内存,IO的使用情况, 如间隔2秒采集3次
```
52_zjdev[/data01/zjgrp/zjdev]%vmstat 2 3
procs -----------memory---------- ---swap-- -----io----       -system--   ------cpu-----
 r  b   swpd   free   buff  cache        si   so    bi    bo        in   cs     us sy id wa st
 0  0 2103292 2772856 570784 44911892    0    0     11    314    0   0           22  4 73  1  0
 8  0 2103292 2772972 570784 44912092    0    0     0     66   45187 109175       2  3 94  0  0
 6  0 2103292 2772476 570784 44912340    0    0     0     500  41664 105258       2  3 94  1  0

进程：
r 运行队列(CPU正在执行或正在等待CPU的进程数)
b 阻塞进程,等待IO的进程数量(不可中断)

内存：
swpd 虚拟内存已使用大小, 如果swpd的值不为0，但是SI，SO的值长期为0，这种情况不会影响系统性能
free 空闲物理内存大小
buff Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存，我本机大概占用500多M
cache cache直接用来记忆我们打开的文件,给文件做缓冲(Linux/Unix的聪明之处，把空闲的物理内存  
的一部分拿来做文件和目录的缓存，是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用)

Swap：
si: 每秒从交换区写到内存的大小，由磁盘调入内存
so: 每秒写入交换区的内存大小，由内存调入磁盘
注意：内存够用的时候，这2个值都是0，如果这2个值长期大于0时，系统性能会受到影响，磁盘IO和CPU资源  
都会被消耗。有些朋友看到空闲内存（free）很少的或接近于0时，就认为内存不够用了，不能光看这一点，  
还要结合si和so，如果free很少，但是si和so也很少（大多时候是0），那么不用担心，系统性能这时不会  
受到影响的

IO(现在的Linux版本块的大小为1kb):
bi: 块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte
bo: 每秒写入的块数
随机磁盘读写的时候，这2个值越大（如超出1024k)，能看到CPU在IO等待的值也会越大

system(系统):
in: 每秒中断数，包括时钟中断
cs: 每秒上下文切换数
上面2个值越大，会看到由内核消耗的CPU时间会越大

CPU(以百分比表示):
us: 用户进程执行时间百分比(us的值比较高时，说明用户进程消耗的CPU时间多，但是如果长期超50%的使用，  
那么我们就该考虑优化程序算法或者进行加速)
sy: 内核系统进程执行时间百分比(sy的值高时，说明系统内核消耗的CPU资源多，这并不是良性表现，我们应该检查原因)
id: 空闲时间百分比
wa: IO等待时间百分比(wa的值高时，说明IO等待比较严重，这可能由于磁盘大量作随机访问造成，也有可能磁盘出现瓶颈,块操作)
st:
```

<!-- more -->

## iostat
查看磁盘I/O使用和CPU情况
```
[root@zhangbb ~]# iostat -d vda 3 3
Linux 3.10.0-693.11.6.el7.x86_64 (zhangbb)      01/08/2019      _x86_64_        (1 CPU)

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               0.36         0.07         3.26    1000024   46518530

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               1.01         0.00         4.04          0         12

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               0.67         0.00        20.20          0         60

tps：该设备每秒的传输次数"一次传输"意思是"一次I/O请求"。多个逻辑请求可能会被合并为"一次I/O请求"。"一次传输"请求的大小是未知的。

kB_read/s：每秒从设备（drive expressed）读取的数据量
kB_wrtn/s：每秒向设备（drive expressed）写入的数据量
kB_read：读取的总数据量
kB_wrtn：写入的总数量数据量；这些单位都为Kilobytes

-x：显示扩展状态
[root@zhangbb ~]# iostat -d -x 1 3
Linux 3.10.0-693.11.6.el7.x86_64 (zhangbb)      01/08/2019      _x86_64_        (1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.44    0.00    0.35     0.07     3.26    18.71     0.00    0.40    1.34    0.39   0.21   0.01

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00

> rrqm/s: 每秒对该设备的读请求被合并次数，文件系统会对读取同块(block)的请求进行合并
> wrqm/s: 每秒对该设备的写请求被合并次数
> r/s: 每秒完成的读次数
> w/s: 每秒完成的写次数
> rkB/s: 每秒读数据量(kB为单位)
> wkB/s: 每秒写数据量(kB为单位)
> avgrq-sz:平均每次IO操作的数据量(扇区数为单位)
> avgqu-sz: 平均等待处理的IO请求队列长度
> await: 平均每次IO请求等待时间(包括等待时间和处理时间，毫秒为单位)
> svctm: 平均每次IO请求的处理时间(毫秒为单位)
> %util: 采用周期内用于IO操作的时间比率，即IO队列非空的时间比率
util = (r/s+w/s) * (svctm/1000)

```

## mpstat
每个CPU的使用情况
```
从/proc/stat获得数据
[root@zhangbb ~]# mpstat 3 5
Linux 3.10.0-693.11.6.el7.x86_64 (zhangbb)      01/08/2019      _x86_64_        (1 CPU)

03:18:09 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
03:18:12 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.33    0.00    0.00   99.67
03:18:15 PM  all    0.34    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   99.66
03:18:18 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
03:18:21 PM  all    0.33    0.00    0.33    0.00    0.00    0.00    0.33    0.00    0.00   99.00
03:18:24 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.34    0.00    0.00   99.66
Average:     all    0.13    0.00    0.07    0.00    0.00    0.00    0.20    0.00    0.00   99.60

CPU 处理器ID
user   在时间段里，用户态的CPU时间(%)，不包含 nice值为负 进程 (usr/total)*100  
nice   在时间段里，nice值为负进程的CPU时间(%)   (nice/total)*100  
system 在时间段里，核心时间(%)   (system/total)*100
iowait 在时间段里，硬盘IO等待时间(%) (iowait/total)*100
irq    在时间段里，硬中断时间(%)      (irq/total)*100
soft   在时间段里，软中断时间(%)    (softirq/total)*100
idle   在时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间(%)(idle/total)*100
intr/s 在时间段里，每秒CPU接收的中断的次数intr/total)*100

```

## free
```
52_zjdev[/data01/zjgrp/zjdev]%free -h
             total       used       free     shared    buffers     cached
Mem:           63G        61G       1.8G        21G       567M        43G
-/+ buffers/cache:        17G        45G
Swap:         2.0G       2.0G        32K

(-buffers/cache) used内存数：第一部分Mem行中的 used – buffers – cached
(+buffers/cache) free内存数: 第一部分Mem行中的 free + buffers + cached
可见-buffers/cache反映的是被程序实实在在吃掉的内存，而+buffers/cache反映的是可以挪用的内存总数
```

## sar
```
sar -u 1 2 //体CPU使用统计
sar -P ALL 1 1 //各个CPU使用统计
sar -r  1 2 //内存使用情况统计
sar -b 1 2 //整体I/O情况
sar -d -p 1 1 //各个I/O设备情况
sar -n DEV 1 1 //网络统计

[root@zhangbb ~]# sar -n DEV 1 1
Linux 3.10.0-693.11.6.el7.x86_64 (zhangbb)      01/08/2019      _x86_64_        (1 CPU)

07:59:15 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
07:59:16 PM      eth0      6.12      5.10      0.59      0.95      0.00      0.00      0.00
07:59:16 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
07:59:16 PM   ip_vti0      0.00      0.00      0.00      0.00      0.00      0.00      0.00
07:59:16 PM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00

IFACE: 网络接口名称
rxpck/s: 每秒收包的数量
txpck/s: 每秒发包的数量
rxkB/s: 每秒收的数据量(kB为单位)
txkB/s: 每秒发的数据量(kB为单位)

sar日志保存(-o)
```

------
# 进程级计数器

## 进程信息

```
/proc
```





## top

按一个统计数据排序，显示排名高的进程
```
52_zjdev[/data01/zjgrp/zjdev]%top -u zjv8cs -d 3
top - 14:16:50 up 168 days, 17:39, 34 users,  load average: 0.95, 0.74, 0.66
Tasks: 418 total,   1 running, 417 sleeping,   0 stopped,   0 zombie
%Cpu(s): 22.1 us,  3.3 sy,  0.0 ni, 73.6 id,  0.7 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem:  66109508 total, 65616660 used,   492848 free,   321260 buffers
KiB Swap:  2103292 total,  2103292 used,        0 free. 47138260 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                  
 3869 zjv8cs    20   0 26.460g 504060  16484 S 6.250 0.762 124:55.11 sframe                                                   
 6343 zjv8cs    20   0 19.333g 332120   7328 S 0.000 0.502  20:45.48 java                                                     
 6369 zjv8cs    20   0 2921872 224360  26232 S 0.000 0.339  77:42.57 odframe                                                  
 6480 zjv8cs    20   0 23.116g 102460  90296 S 0.000 0.155   1:11.94 sframe                                                   
 6482 zjv8cs    20   0 23.064g  27784  16352 S 0.000 0.042   1:09.05 sframe                                                   
 6484 zjv8cs    20   0 23.062g  28872  15584 S 0.000 0.044   1:08.88 sframe                                                   
 6503 zjv8cs    20   0 23.116g  59680  47540 S 0.000 0.090   1:10.93 sframe                                                   
 6511 zjv8cs    20   0 23.119g 358908 336920 S 0.000 0.543   1:22.49 sframe                                                   
 6520 zjv8cs    20   0 23.116g 1.069g 1.056g S 0.000 1.696   1:45.00 sframe                                                   
 6536 zjv8cs    20   0 23.066g  27624  15596 S 0.000 0.042   1:09.88 sframe                                                   
 6543 zjv8cs    20   0 23.288g 2.963g 2.820g S 0.000 4.700   3:10.91 sframe                                                   
 6552 zjv8cs    20   0 23.118g 369656 356788 S 0.000 0.559   1:22.45 sframe                                                   
 6554 zjv8cs    20   0 23.116g  29924  17728 S 0.000 0.045   1:11.02 sframe                                                   
 6567 zjv8cs    20   0 23.061g  26992  15396 S 0.000 0.041   1:09.80 sframe                                                   
 7061 zjv8cs    20   0   31968   1244    736 S 0.000 0.002   0:01.24 systemd                                                  
 7062 zjv8cs    20   0   63640     40      0 S 0.000 0.000   0:00.00 (sd-pam)                                                 
 7492 zjv8cs    20   0 24.637g  62640   7180 S 0.000 0.095  38:41.25 sframe                                                   
 8069 zjv8cs    20   0 23.476g  30552  17872 S 0.000 0.046   9:14.04 odframe                                                  
 8101 zjv8cs    20   0 23.116g  89672  77504 S 0.000 0.136   1:13.43 sframe                                                   
 8156 zjv8cs    20   0 24.643g  68264   6964 S 0.000 0.103  34:18.32 sframe                                                   
 8683 zjv8cs    20   0 23.136g  18508   8916 S 0.000 0.028   1:04.60 sframe                                                   
14603 zjv8cs    20   0 23.003g  28440  16644 S 0.000 0.043   0:24.28 sframe                                                   
14838 zjv8cs    20   0 25.779g 437940  21188 S 0.000 0.662 124:05.00 sframe                                                   
17791 zjv8cs    20   0   87688   1800    904 S 0.000 0.003   0:00.03 sshd                                                     
17792 zjv8cs    20   0   16824   3376   1452 S 0.000 0.005   0:00.15 csh

上半部分：
top一行：从左到右依次为当前系统时间，系统运行的时间，系统在之前1min、5min和15min内cpu的平均负载值
Tasks一行：该行给出进程整体的统计信息，包括统计周期内进程总数、运行状态进程数、休眠状态进程数、停止状态进程数和僵死状态进程数
Cpu(s)一行：cpu整体统计信息，包括用户态下进程、系统态下进程占用cpu时间比，nice值大于0的进程在用户态下占用cpu时间比，cpu处于idle状态、wait状态的时间比，以及处理硬中断、软中断的时间比
Mem一行：该行提供了内存统计信息，包括物理内存总量、已用内存、空闲内存以及用作缓冲区的内存量
Swap一行：虚存统计信息，包括交换空间总量、已用交换区大小、空闲交换区大小以及用作缓存的交换空间大小

下半部：
PID: 进程pid
USER: 拉起进程的用户
PR: 该列值加100为进程优先级，若优先级小于100，则该进程为实时(real-time)进程，  
否则为普通(normal)进程，实时进程的优先级更高，更容易获得cpu调度，以上输出结果中，  
java进程优先级为120，是普通进程，had进程优先级为2，为实时进程，migration 进程的优先级RT对应于0，为最高优先级
NI: 进程的nice优先级值，该列中，实时进程的nice值为0，普通进程的nice值范围为-20~19
VIRT: 进程所占虚拟内存大小（默认单位kB）
RES: 进程所占物理内存大小（默认单位kB）
SHR: 进程所占共享内存大小（默认单位kB）
S: 进程的运行状态
%CPU: 采样周期内进程所占cpu百分比(-d 3)
%MEM: 采样周期内进程所占内存百分比
TIME+: 进程使用的cpu时间总计
COMMAND: 拉起进程的命令

# 可以按照不同的指标排序显示，按对应键即可
# P 按照 CPU 使用率排序
# T 按照 MITE+ 排序
# M 按内存使用占比排序
# 按”d”可以更新top更新频率, 按空格键可以手动更新输出
# 按”c”快捷键，将显示命令的全路径以及命令参数
# 按”k”快捷键，可向指定进程发送信号，默认信号为SIGTERM，该信号可中止进程
# 按”r”快捷键，可以修改指定进程的nice优先级

```

## ps 
```shell
[root@zhangbb ~]# ps -o pid,ppid,lwp,comm
  PID  PPID   LWP COMMAND
 4981  4975  4981 bash
 7792  4981  7792 ps
 
# 进程状态，显示进程的各种统计信息，包括内存和CPU的使用
host-10-19-14-51:/data01/zjgrp/zjv8cs/sqlite> ps -eLo pid,lwp,pcpu | grep 12567
12567 12567  0.0
12567 12568  0.0
12567 12569  0.0
12567 12575  0.0
12567 12576  0.0
12567 12577  0.0
12567 12579  0.0

linux上进程有5种状态: 
1. 运行(正在运行或在运行队列中等待) 
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号) 
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生) 
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放) 
5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行) 

ps工具标识进程的5种状态码: 
D 不可中断 uninterruptible sleep (usually IO) 
R 运行 runnable (on run queue) 
S 中断 sleeping 
T 停止 traced or stopped 
Z 僵死 a defunct (”zombie”) process 

[root@zhangbb ~]# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.4  54384  4916 ?        Ss    2018  85:05 /usr/lib/systemd/systemd --system --deserialize 22
root         2  0.0  0.0      0     0 ?        S     2018   0:03 [kthreadd]
root         3  0.0  0.0      0     0 ?        S     2018   3:55 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<    2018   0:00 [kworker/0:0H]

TTY ：该 process 是在那个终端机上面运作，若与终端机无关，则显示 ?， 
另外， tty1-tty6 是本机上面的登入者程序，若为 pts/0 等等的，则表示为由网络连接进主机的程序
```

## pstree
查看进程中的线程, 一目了然
```
51_zjdev[/data01/zjgrp/zjdev]%pstree -p 5414
server(5414)-+-{server}(5415)
             |-{server}(5416)
             |-{server}(5417)
             |-{server}(5424)
             |-{server}(5425)
             |-{server}(5426)
             |-{server}(5427)
             |-{server}(5428)
             |-{server}(5429)
             `-{server}(5430)
```

## pmap
将进程的内存和使用统计一起列出
```
host-10-19-14-51:/data01/zjgrp/zjv8cs> pmap -d 6480
START               SIZE     RSS     PSS   DIRTY    SWAP PERM OFFSET           DEVICE MAPPING
0000000000400000     28K     24K      1K      0K      0K r-xp 0000000000000000 fd:00  /data01/zjgrp/zjdev/ob_rel/bin/sframe.2.3.0.811124
0000000000606000      4K      4K      4K      4K      0K r--p 0000000000006000 fd:00  /data01/zjgrp/zjdev/ob_rel/bin/sframe.2.3.0.811124
0000000000607000      4K      4K      4K      4K      0K rw-p 0000000000007000 fd:00  /data01/zjgrp/zjdev/ob_rel/bin/sframe.2.3.0.811124
0000000000fbf000    596K    532K    532K    532K      0K rw-p 0000000000000000 00:00  [heap]
0000000001054000   2420K   1980K   1980K   1980K      0K rw-p 0000000000000000 00:00  [heap]
00002ac54f4c6000    132K     96K      0K      0K      0K r-xp 0000000000000000 00:23  /lib64/ld-2.19.so
...
00007f4300ffe000 768064K  72592K  72527K  72592K      0K r--s 0000000000000000 00:05  /SYSV0108cd26
00007f432fe0e000  67528K     12K      0K     12K      0K r--s 0000000000000000 00:05  /SYSV0108ccd5
00007fff0738e000    136K    108K    108K    108K      0K rw-p 0000000000000000 00:00  [stack]
00007fff073b6000      8K      4K      0K      0K      0K r-xp 0000000000000000 00:00  [vdso]
ffffffffff600000      4K      0K      0K      0K      0K r-xp 0000000000000000 00:00  [vsyscall]
Total:           24239152K 102676K  85374K  84936K      0K

49996K writable-private, 1075844K readonly-private, 23113312K shared, and 98852K referenced 

[root@zhangbb ~]# pmap -d 1663
1663:   /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/usr/local/mysql/var/zhangbb.err --open-files-limit=65535 --pid-file=/usr/local/mysql/var/zhangbb.pid --socket=/tmp/mysql.sock --port=3306
Address           Kbytes Mode  Offset           Device    Mapping
0000000000400000    8852 r-x-- 0000000000000000 0fd:00001 mysqld
0000000000ea5000       8 r---- 00000000008a5000 0fd:00001 mysqld
0000000000ea7000     660 rw--- 00000000008a7000 0fd:00001 mysqld
0000000000f4c000     160 rw--- 0000000000000000 000:00000   [ anon ]
0000000001f85000   24832 rw--- 0000000000000000 000:00000   [ anon ]
......
00007f0bd867b000       4 rw--- 0000000000022000 0fd:00001 ld-2.17.so
00007f0bd867c000       4 rw--- 0000000000000000 000:00000   [ anon ]
00007ffc301f0000     132 rw--- 0000000000000000 000:00000   [ stack ]
00007ffc3025b000       8 r-x-- 0000000000000000 000:00000   [ anon ]
ffffffffff600000       4 r-x-- 0000000000000000 000:00000   [ anon ]
mapped: 434848K    writeable/private: 204948K    shared: 0K

mapped 表示该进程映射的虚拟地址空间大小，也就是该进程预先分配的虚拟内存大小，即ps出的vsz
writeable/private  表示进程所占用的私有地址空间大小，也就是该进程实际使用的内存大小     
shared 表示进程和其他进程共享的内存大小
```

## iotop
进程的I/O速度
```
[root@zhangbb ~]# iotop -p 29215
Total DISK READ :       0.00 B/s | Total DISK WRITE :       0.00 B/s
Actual DISK READ:       0.00 B/s | Actual DISK WRITE:       0.00 B/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                        
29215 be/4 root        0.00 B/s    0.00 B/s  ?unavailable?  docker-containerd -l uni~him --runtime docker

左右箭头：改变排序方式，默认是按IO排序。
r：改变排序顺序。
o：只显示有IO输出的进程。
p：进程/线程的显示方式的切换。
a：显示累积使用量。
q：退出。
```

## pidstat
查看进程I/O , 内存, CPU
```
# cpu使用情况统计(-u)
# 使用-u选项，pidstat将显示各活动进程的cpu使用统计，执行”pidstat -u”与单独执行”pidstat”的效果一样
[root@zhangbb ~]# pidstat
Linux 3.10.0-693.11.6.el7.x86_64 (zhangbb)      01/09/2019      _x86_64_        (1 CPU)

03:11:40 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
03:11:40 PM     0         1    0.01    0.02    0.00    0.03     0  systemd
03:11:40 PM     0         2    0.00    0.00    0.00    0.00     0  kthreadd
03:11:40 PM     0         3    0.00    0.00    0.00    0.00     0  ksoftirqd/0
03:11:40 PM     0         9    0.00    0.01    0.00    0.01     0  rcu_sched

03:11:40: pidstat获取信息时间点
PID: 进程pid
%usr: 进程在用户态运行所占cpu时间比率
%system: 进程在内核态运行所占cpu时间比率
%CPU: 进程运行所占cpu时间比率
CPU: 指示进程在哪个核运行
Command: 拉起进程对应的命令

# 内存使用情况统计(-r)
[root@zhangbb ~]# pidstat -r -p 31846
Linux 3.10.0-693.11.6.el7.x86_64 (zhangbb)      01/14/2019      _x86_64_        (1 CPU)

02:46:04 PM   UID       PID  minflt/s  majflt/s     VSZ    RSS   %MEM  Command
02:46:04 PM     0     31846      0.00      0.00  216704   2924   0.29  osqueryd

minflt/s: 每秒次缺页错误次数(minor page faults)，次缺页错误次数意即虚拟内存地址映射成物理内存地址产生的page fault次数
majflt/s: 每秒主缺页错误次数(major page faults)，当虚拟内存地址映射成物理内存地址时，相应的page在swap中，这样的page fault为major page fault，一般在内存使用紧张时产生
VSZ: 该进程使用的虚拟内存(以kB为单位)
RSS: 该进程使用的物理内存(以kB为单位)
%MEM: 该进程使用内存的百分比
Command: 拉起进程对应的命令

# IO情况统计(-d)
[root@zhangbb ~]# pidstat -d 1 2
Linux 3.10.0-693.11.6.el7.x86_64 (zhangbb)      01/14/2019      _x86_64_        (1 CPU)

02:47:23 PM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command

02:47:24 PM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command

Average:      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command

kB_rd/s: 每秒进程从磁盘读取的数据量(以kB为单位)
kB_wr/s: 每秒进程向磁盘写的数据量(以kB为单位)
Command: 拉起进程对应的命令

# 针对特定进程统计(-p)

pidstat -T CHILD -C mysql // 显示所有mysql服务器的子进程
pidstat -urd -h // 将所有的统计数据结合到一个便于阅读的单一报告中
```

## fuser
[常用命令](https://www.cnblogs.com/bangerlee/tag/linux/)
查询给定文件或目录的用户或进程信息
```
[root@zhangbb ~]# fuser -v /root
                     USER        PID ACCESS COMMAND
/root:               root       7944 ..c.. bash
                     root       9847 ..c.. bash
                     root      25352 ..c.. bash
					 
PID后跟的字符说明了进程以何种方式与该目录/文件关联，有以下关联方式：
　　c  指示进程的工作目录
　　e  指示该文件为进程的可执行文件(即进程由该文件拉起)
　　f  指示该文件被进程打开，默认情况下f字符不显示
　　F  指示该文件被进程打开进行写入，默认情况下F字符不显示
　　r  指示该目录为进程的根目录
　　m  指示进程使用该文件进行内存映射，抑或该文件为共享库文件，被进程映射进内存

[root@zhangbb ~]# fuser -v -n tcp 3306
                     USER        PID ACCESS COMMAND
3306/tcp:            mysql      1663 F.... mysqld
查询socket 端口占用进程

fuser -v -k mysqld
关闭进程
```

------
# 网络工具

## netstat
```
netstat -a     #列出所有端口
netstat -at    #列出所有tcp端口
netstat -au    #列出所有udp端口  

netstat -l        #只显示监听端口
netstat -lt       #只列出所有监听 tcp 端口
netstat -lu       #只列出所有监听 udp 端口

netstat -pt    #显示 PID 和进程名称
netstat -n     #显示IP
netstat -c   #每隔一秒输出网络信息
```

## ss
```
比netstat 效率高, 可以用time来比较
-a, --all            display all sockets
-l, --listening      display listening sockets
-p, --processes      show process using socket
-s, --summary        show socket usage summary
-t, --tcp            display only TCP sockets
-u, --udp            display only UDP sockets

#列出来至120.33.31.1,80端口的连接
ss src 120.33.31.1:http
ss src 120.33.31.1:80

host-10-19-14-51:/data01/zjgrp/zjv8cs> ss -s
Total: 2701 (kernel 0)
TCP:   2018 (estab 785, closed 1049, orphaned 0, synrecv 0, timewait 16/0), ports 0

Transport Total     IP        IPv6
*         0         -         -        
RAW       0         0         0        
UDP       1         1         0        
TCP       969       831       138      
INET      970       832       138      
FRAG      0         0         0     
```

## iftop
```
用来监控网卡的实时流量（可以指定网段）、反向解析IP、显示端口信息等

                      12.5Kb                25.0Kb                37.5Kb                50.0Kb          62.5Kb
+---------------------+---------------------+---------------------+---------------------+---------------------
45.32.49.180.vultr.com                    => 218.205.54.1                              6.88Kb  5.00Kb  5.00Kb
                                          <=                                            528b    424b    424b 
45.32.49.180.vultr.com                    => 108.61.10.10.choopa.net                    472b   2.36Kb  2.36Kb
                                          <=                                              0b   2.80Kb  2.80Kb
zhangbb                                   => 2605:a000:1620:43be:bc3d:cf55:9cc5:c949   1.67Kb   856b    856b 
                                          <=                                            608b    304b    304b
zhangbb                                   => 2605:a000:1102:c230:b272:bfff:fe82:223f      0b    856b    856b
                                          <=                                              0b    304b    304b
zhangbb                                   => 2001:b011:200b:1d2c:cdd3:eb53:ee38:be79   1.63Kb   834b    834b
                                          <=                                            584b    292b    292b
zhangbb                                   => 2001:e68:541b:b09:ddb2:79ab:d9f2:a2fb     1.63Kb   834b    834b
                                          <=                                            584b    292b    292b
45.32.49.180.vultr.com                    => 212.92.105.127                             640b    320b    320b
                                          <=                                            528b    264b    264b
45.32.49.180.vultr.com                    => 188.168.215.41                               0b    308b    308b
                                          <=                                              0b    252b    252b
45.32.49.180.vultr.com                    => m5673.contaboserver.net                      0b    306b    306b
                                          <=                                              0b    250b    250b
45.32.49.180.vultr.com                    => ip-187-44-249-173.static.contabo.net         0b    302b    302b
                                          <=                                              0b    246b    246b
zhangbb                                   => 2a01:c50f:8c01:2700:8cfe:47e7:f6f3:41c       0b    202b    202b
                                          <=                                              0b    238b    238b
zhangbb                                   => 2400:2412:8720:2800:7603:bdff:fe96:676b    424b    212b    212b
                                          <=                                            388b    194b    194b
zhangbb                                   => 2607:5300:60:5e6e::33c:e975                424b    212b    212b
                                          <=                                            388b    194b    194b
zhangbb                                   => 240d:1a:700:e900:211:32ff:fe69:b9be          0b    194b    194b
                                          <=                                              0b    212b    212b
--------------------------------------------------------------------------------------------------------------
TX:             cum:   6.56KB   peak:   14.4Kb                                rates:   14.4Kb  13.1Kb  13.1Kb
RX:                    3.19KB           9.05Kb                                         3.71Kb  6.38Kb  6.38Kb
TOTAL:                 9.76KB           20.9Kb                                         18.1Kb  19.5Kb  19.5Kb

TX：发送流量
RX：接收流量
TOTAL：总流量
Cumm：运行iftop到目前时间的总流量
peak：流量峰值
rates：分别表示过去 2s 10s 40s 的平均流量
```

## nethogs
一个小型的net top工具, 按进程或程序实时统计网络带宽使用率
```
[root@zhangbb ~]# nethogs
Ethernet link detected
                      Waiting for first packet to arrive (see sourceforge.net bug 1019381)
NetHogs version 0.8.5

    PID USER     PROGRAM                                                  DEV        SENT      RECEIVED       
   2790 root     sshd: root@pts/0,pts/1                                   eth0        1.419       0.555 KB/sec
      ? root     unknown TCP                                                          0.000       0.000 KB/sec

  TOTAL                                                                               1.419       0.555 KB/sec

m : Cycle between display modes (kb/s, kb, b, mb) 切换网速显示单位
r : Sort by received. 按接收流量排序
s : Sort by sent. 按发送流量排序
q : Quit and return to the shell prompt. 退出NetHogs命令工具
```

## tpcdump
```
// 监视指定网络接口的数据包
tcpdump -i eth1
// 截获主机210.27.48.1 和主机210.27.48.2 或210.27.48.3的通信
tcpdump host 210.27.48.1 and \ (210.27.48.2 or 210.27.48.3 \) 
// 取主机210.27.48.1除了和主机210.27.48.2之外所有主机通信的ip包
tcpdump ip host 210.27.48.1 and ! 210.27.48.2
// 截获主机hostname发送的所有数据
tcpdump -i eth0 src host hostname
// 监视所有送到主机hostname的数据包
tcpdump -i eth0 dst host hostname

// 监视指定主机和端口的数据包
tcpdump tcp port 23 and host 210.27.48.1
tcpdump udp port 123 

// 监视指定协议的数据包
// 打印TCP会话中的的开始和结束数据包, 并且数据包的源或目的不是本地网络上的主机.
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'
......

// 保存到文件用wireshark分析
tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap
(1)tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
(2)-i eth1 : 只抓经过接口eth1的包
(3)-t : 不显示时间戳
(4)-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
(5)-c 100 : 只抓取100个数据包
(6)dst port ! 22 : 不抓取目标端口是22的数据包
(7)src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24
(8)-w ./target.cap : 保存成cap文件，方便用ethereal(即wireshark)分析

```

[Linux tcpdump命令详解](https://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html)



## iperf

测量 TCP 和 UDP 的网络性能；测量最大带宽，并汇报延时和数据报的丢失情况





------
# 其他命令

## tar
使用 tar 命令只要记得参数是[必选+自选+f]:
必选：
1. -c 意为 create，表示创建压缩包
2. -x 意为 extract，表示解压
3. -t 表示查看内容
4. -r 给压缩包追加文件
5. -u 意为 update，更新压缩包中的文件

自选：
1. -z 使用 gzip 属性
2. -j 使用 bz2 属性
3. -Z 使用 compress 属性
4. -v 意为 verbose，显示详细的操作过程
5. -O 将文件输出到标准输出

遇到不同类型的文件，请用不同的套路来应对：
> *.tar -> tar -xf
> *.tar.gz -> tar -xzf
> *.tar.bz2 -> tar -xjf
> *.tar.Z -> tar -xZf
> *.gz -> gzip -d
> *.rar -> unrar e
> *.zip -> unzip

## find
```
# 找出七天前的文件
# / 表示从根目录中查找
# -type f 表示找出系统普通文件，不包含目录
# -mtime +n 表示寻找 n 天前的数据
# -print 打印文件名称
find / -type f -mtime +7 -print

# 找出并删除七天前的文件
find /temp/ -type f -mtime +7 -print -exec rm -f {} \;
# -exec 表示后面执行系统命令
# {} 只有该符号能跟在命令你后面
# \; 结束符号
find /temp/ -type f -mtime +7 -print | xargs rm -f
# 使用管道和 xargs = -exec

# 查找 /var 下最大的十个文件
find /var -type f -ls | sort -k 7 -r -n | head

# 查找 /var/log 下大于 5GB 的文件
find /var/log/ -type f -size +5120M -exec ls -lh {} \;

# 找出今天所有文件并将它们拷贝到另一个目录
find . -ctime 0 -print -exec cp {} /mnt/backup/{} \;

# 找出2天内的修改文件, 删除前询问
find /var/log -type f -mtime -2 -ok rm {} \;

#-cmin +10: 在10分钟前被修改过的

```

## du
```
#显示当前文件下 Top 10 空间占用的文件/目录，
#s 表示不显示每个子目录或文件的大小
#h 表示用更加自然的方式显示（比如 K/M/G 这样）
du -sh * | sort -nr | head
```

## dd
测试磁盘读写速度, 备份或恢复磁盘
```
参数：
b=512, c=1, k=1024, w=2, xm=number m  指定数字的地方若以下列字符结尾乘以相应的数字
if=file 输入文件名，缺省为标准输入
of=file 输出文件名，缺省为标准输出
ibs=bytes 一次读入 bytes 个字节(即一个块大小为 bytes 个字节
obs=bytes 一次写 bytes 个字节(即一个块大小为 bytes 个字节
bs=bytes 同时设置读写块的大小为 bytes ，可代替 ibs 和 obs
cbs=bytes 一次转换 bytes 个字节，即转换缓冲区大小
skip=blocks 从输入文件开头跳过 blocks 个块后再开始复制
seek=blocks 从输出文件开头跳过 blocks 个块后再开始复制
count=blocks 仅拷贝 blocks 个块，块大小等于 ibs 指定的字节数
应用：
// 将/dev/hdx全盘数据备份到指定路径的image文件
dd if=/dev/hdx of=/path/to/image
// 备份/dev/hdx全盘数据，并利用gzip工具进行压缩，保存到指定路径
dd if=/dev/hdx | gzip >/path/to/image.gz

// 拷贝内存资料到硬盘, 将内存里的数据拷贝到root目录下的mem.bin文件
dd if=/dev/mem of=/root/mem.bin bs=1024
// 从光盘拷贝iso镜像, 从光盘拷贝iso镜像
dd if=/dev/cdrom of=/root/cd.iso
// 备份文件恢复到指定盘
dd if=/path/to/image of=/dev/hdx
gzip -dc /path/to/image.gz | dd of=/dev/hdx
// 备份MBR
dd if=/dev/hdx of=/path/to/image count=1 bs=512
// 将内存里的数据拷贝到root目录下的mem.bin文件
dd if=/dev/mem of=/root/mem.bin bs=1024
// 拷贝光盘数据到root文件夹下，并保存为cd.iso文件
dd if=/dev/cdrom of=/root/cd.iso
// 利用随机的数据填充硬盘，在某些必要的场合可以用来销毁数据
dd if=/dev/urandom of=/dev/hda1

// 得到最恰当的block size
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
dd if=/dev/zero bs=2048 count=500000 of=/root/1Gb.file
dd if=/dev/zero bs=4096 count=250000 of=/root/1Gb.file      
dd if=/dev/zero bs=8192 count=125000 of=/root/1Gb.file
// 测试硬盘读写速度
dd if=/root/1Gb.file bs=64k | dd of=/dev/null
dd if=/dev/zero of=/root/1Gb.file bs=1024 count=1000000

[root@zhangbb ~]# dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
1000000+0 records in
1000000+0 records out
1024000000 bytes (1.0 GB) copied, 4.31042 s, 238 MB/s
[root@zhangbb ~]# dd if=/root/1Gb.file bs=64k | dd of=/dev/null
15625+0 records in
15625+0 records out
1024000000 bytes (1.0 GB) copied2000000+0 records in
2000000+0 records out
1024000000 bytes (1.0 GB) copied, 3.08384 s, 332 MB/s, 3.08522 s, 332 MB/s
```

## lsof
```
# 查看进程打开文件
lsof -p pid

[root@zhangbb ~]# lsof -p `pgrep pritunl-web`
COMMAND     PID USER   FD      TYPE   DEVICE SIZE/OFF     NODE NAME
pritunl-w 28661 root  cwd       DIR    253,1     4096        2 /
pritunl-w 28661 root  rtd       DIR    253,1     4096        2 /
pritunl-w 28661 root  txt       REG    253,1 13205960    46464 /usr/bin/pritunl-web
pritunl-w 28661 root  mem       REG    253,1  2151672     4698 /usr/lib64/libc-2.17.so
pritunl-w 28661 root  mem       REG    253,1   141968     4724 /usr/lib64/libpthread-2.17.so
pritunl-w 28661 root  mem       REG    253,1   163400     4690 /usr/lib64/ld-2.17.so
pritunl-w 28661 root    0r      CHR      1,3      0t0     4856 /dev/null
pritunl-w 28661 root    1w     FIFO      0,8      0t0 62148038 pipe
pritunl-w 28661 root    2w     FIFO      0,8      0t0 62148039 pipe
pritunl-w 28661 root    3w      REG    253,1      141    46468 /var/log/pritunl.log
pritunl-w 28661 root    4u     IPv6 62148054      0t0      TCP *:https (LISTEN)
pritunl-w 28661 root    5u  a_inode      0,9        0     4852 [eventpoll]
pritunl-w 28661 root   10r     FIFO      0,8      0t0 62147876 pipe
pritunl-w 28661 root   11r      CHR      1,9      0t0     4861 /dev/urandom
pritunl-w 28661 root   12w      CHR      1,3      0t0     4856 /dev/null
pritunl-w 28661 root   13w     FIFO      0,8      0t0 62147876 pipe
pritunl-w 28661 root   14u     sock      0,7      0t0 62147877 protocol: NETLINK

```

## curl

## iptables

## ipcs
用于显示进程间通信信息的命令，其可列出进程间的共享内存、消息队列、信号信息

```
# 
host-10-19-14-51:/data01/zjgrp/zjv8cs> ipcs -a

--------- 消息队列 -----------
键         msqid      拥有者     权限       已用字节数   消息        
0x0000bff0 688128     zjv8cs2    666        0            0           
0x000cd141 1605633    zjdev      666        0            0           

------------ 共享内存段 --------------
键         shmid      拥有者     权限       字节       连接数     状态        
0x0100cd0c 55115778   zjv8cs     666        1761104    44                      
0x0108ccd5 55148547   zjv8cs     666        69145336   21                      
0x0108cd26 55181317   zjv8cs     666        786497536  21                                                              
0x0108ca0b 55214104   zjv8cs     666        786497536  21                                                        
0x0108ca30 55246915   zjv8cs     666        786497536  21                      
0x0108ca3c 55279684   zjv8cs     666        786497536  21                      
0x0108cbe2 55312453   zjv8cs     666        786497536  21                      
0x0108cd4a 55345222   zjv8cs     666        786497536  21                      
0x0108ce69 55377991   zjv8cs     666        786497536  21                      
0x0108cf7d 55410760   zjv8cs     666        786497536  21                      
0x0108d3ea 55443529   zjv8cs     666        786497536  21                      
0x0108d4f2 55476298   zjv8cs     666        786497536  21                      
0x0108d4f3 55509067   zjv8cs     666        786497536  21                      
0x0108d4f4 55541836   zjv8cs     666        786497536  21                                    
                               
0x0108d4f5 55574627   zjv8cs     666        786497536  21                      
0x0108d4f6 55607396   zjv8cs     666        786497536  21                      
0x0108d4f7 55640165   zjv8cs     666        786497536  21                                         
0x0108d4f8 55672946   zjv8cs     666        786497536  21                      
0x0108d4f9 55705715   zjv8cs     666        786497536  21                      
0x0108d4fa 55738484   zjv8cs     666        786497536  21                      
0x0108d4fb 55771253   zjv8cs     666        786497536  21                      
0x0108d4fc 55804022   zjv8cs     666        786497536  21                      
0x0108d4fd 55836791   zjv8cs     666        786497536  21                      
0x0108d4fe 55869560   zjv8cs     666        786497536  21                      
0x0108d4ff 55902329   zjv8cs     666        786497536  21                      
0x0108d500 55935098   zjv8cs     666        786497536  21                      
0x0108d501 55967867   zjv8cs     666        786497536  21                      
0x0108d502 56000636   zjv8cs     666        786497536  21                      
0x0108d503 56033405   zjv8cs     666        786497536  21                      
0x0108d504 56066174   zjv8cs     666        786497536  21                      
0x0108d505 56098943   zjv8cs     666        786497536  21                      
0x0108d506 56131712   zjv8cs     666        786497536  21                                       

--------- 信号量数组 -----------
键         semid      拥有者     权限       nsems     
0x0108cd24 3571715    zjv8cs     666        100         


# 查看ipc限制
# -m选项单独列出共享内存信息，-s选项单独列出信号信息，-q选项单独列出消息队列信息
host-10-19-14-51:/data01/zjgrp/zjv8cs> ipcs -m -l

---------- 同享内存限制 ------------
最大段数 = 4096
最大段大小 (千字节) = 18014398509481983
最大总共享内存 (千字节) = 18014398509480960
最小段大小 (字节) = 1

# 显示ipc信息详情
host-10-19-14-51:/data01/zjgrp/zjv8cs> ipcs -m -i 55115778

共享内存段 shmid=55115778
uid=1002        gid=1002        cuid=1002       cgid=100
模式=0666       访问权限=0666
字节数=1761104  lpid=7279       cpid=6369       nattch=48
附加时间=Wed Jan 16 11:38:29 2019  
脱离时间=Wed Jan 16 11:38:29 2019  
更改时间=Wed Jan  2 10:09:05 2019  

# 显示ipc创建者与拥有者详情

# 显示最近访问ipc的进程pid
host-10-19-14-51:/data01/zjgrp/zjv8cs> ipcs -m -p 

-------- 共享内存 创建者/上次修改者 PID ----------
shmid      拥有者     cpid       lpid      
0          zjdev      3140       3783      
6979585    zjcccs     20087      3047      
55115778   zjv8cs     6369       8573      
55148547   zjv8cs     6367       9191      

# 显示最近访问时间点
host-10-19-14-51:/data01/zjgrp/zjv8cs> ipcs -s -t

------------ 信号量 操作/更改 时间 -------------
semid    拥有者     上一操作                   上次更改                  
3538944  zjv8cs2     Wed Jan 16 14:00:16 2019   Wed Jan 16 13:15:46 2019  
524289   zjcccs      Mon Nov 19 14:29:06 2018   Thu Jul 26 17:01:22 2018  
393218   zjdev       Tue Jan  8 16:21:57 2019   Thu Nov 15 18:00:31 2018  
3571715  zjv8cs      Wed Jan 16 14:00:15 2019   Wed Jan  2 10:25:00 2019  
589828   zjv5cs      Wed Jan  9 10:57:59 2019   Thu Jul 26 20:42:16 2018  
950277   zjcccs2     Sun Jan 13 17:08:26 2019   Tue Aug  7 15:35:27 2018  
2228230  zjdev       Thu Oct 25 11:40:30 2018   Thu Oct 25 11:39:07 2018  
2031623  zjcccs3     Wed Jan 16 12:49:44 2019   Thu Sep 27 11:25:15 2018 

# 显示当前使用状态
host-10-19-14-51:/data01/zjgrp/zjv8cs> ipcs -u

---------- 消息状态 -----------
已分配队列数 = 2
已用消息头(header)数 = 0
已用空间 = 0 字节

---------- 共享内存状态 ------------
段已分配 135
页已分配 31145111
页驻留  5595656
页交换  108302
交换性能：0 次尝试       0 次成功

--------- 信号量状态 -----------
已使用数组 = 8
已分配信号量数 = 561

# 删除
ipcrm用法 
ipcrm -M shmkey  移除用shmkey创建的共享内存段
ipcrm -m shmid    移除用shmid标识的共享内存段
ipcrm -Q msgkey  移除用msqkey创建的消息队列
ipcrm -q msqid  移除用msqid标识的消息队列
ipcrm -S semkey  移除用semkey创建的信号
ipcrm -s semid  移除用semid标识的信号

```

## tee
tee命令用于将数据重定向到文件

```
sudo tee /etc/yum.repos.d/mongodb-org-4.0.repo << EOF
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
EOF

[root@zhangbb ~]# cat /etc/yum.repos.d/mongodb-org-4.0.repo
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

# 常用技巧：
```
cat file1 file2 >file3  #合并文件
tac file #以行为单位，倒序显示
head -n 100 file #显示file的前100行
head -n -100 file #显示file的除最后100行以外的内容。  
tail -100 file  #显示file最后100行内容
tail -n +100 file  #从第100行开始显示file内容   
sed -n '1,100p' file > file2 #截取file中1到100行到file2
sed -n "&&&&&&&" file #显示包含"&&&&&"行
sort/uniq #文本排序
date +%s #将当前时间转成Unix时间戳
date -d '2013-2-22 22:14' +%s #指定日期转成Unix时间戳
date -d @1361542596 +"%Y-%m-%d %H:%M:%S" #指定格式输出
date +"%s" # 显示当前时间的Unix时间戳
grep 'aaa\|bbb' file  #查询&&&或$$$
grep -v 'root' file #查询不包含root的行
ps -ef | grep gmake | grep –v root | awk '{print $2}' | xargs  kill -9 # 杀进程
echo "password" | passwd testuser --stdin # linux用shell修改密码
cut -d " " -f 2,3 1.txt # 用空格分隔, 显示第二、三个字段列
cat /etc/*-release # 查看系统版本
ls -ld test/ # 查看目录权限, r/w 读/写取目录结构和目录里的文件, x 表示进入该目录的权限
dmesg # 打印内核的消息缓冲区的信息
cat /proc/cpuinfo # 查看cpu信息
cat /etc/passwd # 查看用户组, useradd -d dir -g group -m username
cat /etc/group # 查看组信息, groupadd -r/-g zjgrp, id/groups
w # 查看当前登录用户
```

# 其他
[awk](https://my.oschina.net/u/4007037/blog/3063882)
[ps/pmap/top等工具源码](https://gitlab.com/procps-ng/procps/blob/master/README.md)



# 找出最大CPU线程

```shell
host-10-19-14-51:/data01/zjgrp/zjv8cs2/users> ps -Leo pid,lwp,user,comm,pcpu|awk '$4=="sframe"{print $0}'|sort -k5 -r -n|head --lines 3
31953 32248 zjv8cs2  sframe           3.8
31953 32169 zjv8cs2  sframe           3.0
30847 30912 zjv8cs2  sframe           1.0

host-10-19-14-51:/data01/zjgrp/zjv8cs2/users> gstack 31953 > 31953_gstack.log

host-10-19-14-51:/data01/zjgrp/zjv8cs2/users> cat 31953_gstack.log |grep 32248
Thread 5 (Thread 0x2ae62bff7700 (LWP 32248)):

```


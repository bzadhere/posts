---
title: linux性能优化之磁盘文件 笔记
date: 2019-02-28 19:25:01
tags:
categories: 读书笔记
---

# 文件系统
> 磁盘为系统提供了最基本的持久化存储
> 文件系统则在磁盘的基础上, 提供了一个用来管理文件的树状结构

## 索引节点和目录
> 索引节点(inode), 用来记录文件的元数据, 比如inode编号, 文件大小, 访问权限 
修改日期, 数据位置等。它和文件一一对应, 跟文件内容一样, 会被持久化到磁盘, 也会占磁盘空间

> 目录项(dentry), 用来记录文件名字, 索引节点指针以及和其他目录项的关系。 
多个关联目录项构成文件系统的目录结构, 由内存维护的内存数据结构, 也叫目录缓存

![](linux性能优化之磁盘文件-笔记/1.png)
<!-- more -->
磁盘格式化时被分为三个区
* 超级块, 存储整个文件系统的状态
* 索引节点区, 存储inode
* 数据存储区, 存储文件数据



## 虚拟文件系统
linux系统文件系统四大基本要素, 目录项/索引节点/超级块/数据块区
为支持不同文件系统, linux内核在用户进程和文件系统之间, 抽象出了一层虚拟文件系统(VFS)。
VFS定义了一组所有文件系统都支持的数据结构和标准接口。

![](linux性能优化之磁盘文件-笔记/2.png)

## 文件系统I/O
VFS提供了一组文件访问接口。文件读写I/O的四种方式分类, 缓冲/非缓冲, 直接/非直接, 阻塞/非阻塞, 同步/异步

> 缓冲IO, 利用标准库缓存来加速文件访问, 标准库内部通过系统调度来访问文件(printf遇到换行符输出)
> 非缓冲IO, 直接通过系统调用来访问文件, 不再经过标准库缓存

> 直接IO, 跳过操作系统页缓存, 直接和文件系统交互访问文件(数据库写)
> 非直接IO, 文件读写时, 先要经过系统的页缓存, 再由内核或系统写入磁盘

> 阻塞IO, 执行操作后没有获得响应, 阻塞当前线程, 不会执行其他任务(管道或网络套接字O_NONBLOCK)
> 非阻塞IO, 执行操作后不会阻塞当前线程, 可以执行其他程序, 通过轮询获取响应结果(select/poll)

> 同步IO, 执行IO操作后, 一直等到整个IO完成后, 才获得结果响应(O_SYNC文件数据和元数据写入磁盘)
> 异步IO, 执行IO操作后, 不用等待完成和完成后响应, 可以继续执行。IO完成后通过事件方式通知应用程序

阻塞和非阻塞/同步和异步 是从两个不同角度划分, 描述的对象也不同, 分别是应用程序和系统。

## 查看工具

```
[root@localhost ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   17G  7.5G  9.5G  44% /
devtmpfs                 486M     0  486M   0% /dev
tmpfs                    497M     0  497M   0% /dev/shm
tmpfs                    497M  6.6M  490M   2% /run
tmpfs                    497M     0  497M   0% /sys/fs/cgroup
/dev/sda1               1014M  142M  873M  14% /boot
tmpfs                    100M     0  100M   0% /run/user/0

[root@localhost ~]# df -ih
Filesystem              Inodes IUsed IFree IUse% Mounted on
/dev/mapper/centos-root   8.5M  101K  8.4M    2% /
devtmpfs                  122K   361  122K    1% /dev
tmpfs                     125K     1  125K    1% /dev/shm
tmpfs                     125K   422  124K    1% /run
tmpfs                     125K    16  125K    1% /sys/fs/cgroup
/dev/sda1                 512K   328  512K    1% /boot
tmpfs                     125K     1  125K    1% /run/user/0

// 索引节点占磁盘容量, 在磁盘格式化时生成好的。磁盘空间充足, 索引节点空间不足 
// 可能是小文件过多导致

// 内核使用 Slab 机制，管理目录项和索引节点的缓存
 cat /proc/slabinfo | grep -E '^#|dentry|inode' 
```

## 总结
为了降低慢速磁盘对性能的影响, 文件系统通过页缓存, 目录项缓存以及所有节点缓存, 
缓和磁盘延迟对应用的影响

# 磁盘I/O
## 磁盘
磁盘是持久化存储设备, 根据存储介质不同, 分为机械磁盘和固态磁盘.
机械磁盘的最小读写扇区, 一般大小为512字节; 固态磁盘的最小读写单位是页, 通常大小是4K/8K

磁盘按接口分类, IDE/SCSI/SAS/SATA/FC. 不同接口分配不同设备名称, IDE->hd, SCSI/SATA->sd 
有多块同类型的磁盘时, 会按a,b,c等字母数序来编号

磁盘根据需要, 分为不同的区, 如/dev/sda1, /dev/sda2

另一种比较常用的架构, 磁盘阵列(RAID); 或者网络存储集群, 通过NFS, SMB, ISCSI等网络存储协议 
暴露给服务器使用。

## 通用块层
在linux中, 磁盘作为一个块设备来管理. 为了减少不同块设备差异带来的影响, linux通过一个统一的 
通用块层来管理, 处于磁盘驱动和文件系统之间的一个块设备抽象层.

主要功能：
> 向上, 为文件系统和应用提供标准接口; 向下, 把各种异构块设备抽象, 提供统一框架管理
> 对I/O请求排序合并, 提高磁盘读写效率

## I/O栈
* 文件系统层, 包括VFS和其他文件系统的具体体现,提供标准接口访问文件
* 通用块层, 包括块设备I/O队列和I/O调度器, 对I/O请求排序合并, 然后发送给下一层设备
* 设备层, 包括存储设备和相应驱动程序, 负责最终物理设备的I/O操作


## 磁盘性能指标
> 使用率, 磁盘处理I/O的时间百分比
> 饱和度, 磁盘I/O的繁忙程度, 100%时无法再接受新的请求
> IOPS, 每秒I/O请求数
> 吞吐量, 每秒的I/O请求大小
> 响应时间, I/O请求从发出到收到响应的时间间隔

## 查看工具
iostat->iotop->pidstat
```
[root@localhost ~]# iostat -d -x 1
Linux 3.10.0-693.el7.x86_64 (localhost)         03/04/2019      _x86_64_        (1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.01    0.28    0.11    15.38     1.14    83.63     0.00   11.81    8.90   19.28   2.47   0.10
dm-0              0.00     0.00    0.26    0.11    14.69     1.04    84.72     0.00   11.39    7.51   20.90   1.93   0.07
dm-1              0.00     0.00    0.01    0.00     0.19     0.00    55.94     0.00   36.47   36.47    0.00  34.90   0.02

// await 平均每次设备 I/O 操作的等待时间(毫秒)
// avgqu-sz 平均 I/O 队列长度
// avgrq-sz 平均每次设备 I/O 操作的数据大小(扇区)
```
![](linux性能优化之磁盘文件-笔记/5.png)

* util% 磁盘I/O使用率
* r/s+w/s IOPS
* rkB/s+wkB/s 是吞吐量
* r_await+w_await 


```
[root@localhost ~]# pidstat -d 1
Linux 3.10.0-693.el7.x86_64 (localhost)         03/04/2019      _x86_64_        (1 CPU)

07:45:24 AM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command

kB_rd/s: 每秒进程从磁盘读取的数据量(以kB为单位)
kB_wr/s: 每秒进程向磁盘写的数据量(以kB为单位)
Command: 拉起进程对应的命令
```


## 案例分析：找出不停打印进程

top：cpu的wa很高
iostat: util% IO使用率高, 请求队列长
pidstat -d : 找出进程, 理解应该用iotop
strace -pf pid :f 跟踪由fork调用所产生的子进程 , 确认具体调用系统函数
lsof -p : 查看打开的文件

进程I/O高, 直接top+lsof解决

## 案例分析：sql执行慢
```
top+ iostat+ pidstat 确定mysql慢

$ lsof -p 28014

// 上一条命令退出时的返回值, 0表示成功, 1表示失败
$ echo $?
1

// -t 表示显示线程，-a 表示显示命令行参数
$ pstree -t -a -p 27458
mysqld,27458 --log_bin=on --sync_binlog=1
...
  ├─{mysqld},27922
  ├─{mysqld},27923
  └─{mysqld},28014

// lsof 需要进程号
$ lsof -p 27458
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
...
​mysqld  27458      999   38u   REG    8,1 512440000 2601895 /var/lib/mysql/test/products.MYD

// strace 确定在做什么
$ strace -f -p 27458
[pid 28014] read(38, "934EiwT363aak7VtqF1mHGa4LL4Dhbks"..., 131072) = 131072
[pid 28014] read(38, "hSs7KBDepBqA6m4ce6i6iUfFTeG9Ot9z"..., 20480) = 20480
[pid 28014] read(38, "NRhRjCSsLLBjTfdqiBRLvN9K6FRfqqLm"..., 131072) = 131072
[pid 28014] read(38, "AKgsik4BilLb7y6OkwQUjjqGeCTQTaRl"..., 24576) = 24576
[pid 28014] read(38, "hFMHx7FzUSqfFI22fQxWCpSnDmRjamaW"..., 131072) = 131072
[pid 28014] read(38, "ajUzLmKqivcDJSkiw7QWf2ETLgvQIpfC"..., 20480) = 20480

//  显示mysql使用目录, 只是为了进一步确认
$ docker exec -i -t mysql mysql -e 'show global variables like "%datadir%";'
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+

mysql> show full processlist;
+----+------+-----------------+------+---------+------+--------------+-----------------------------------------------------+
| Id | User | Host            | db   | Command | Time | State        | Info                                                |
+----+------+-----------------+------+---------+------+--------------+-----------------------------------------------------+
| 27 | root | localhost       | test | Query   |    0 | init         | show full processlist                               |
| 28 | root | 127.0.0.1:42262 | test | Query   |    1 | Sending data | select * from products where productName='geektime' |
+----+------+-----------------+------+---------+------+--------------+-----------------------------------------------------+
2 rows in set (0.00 sec)

# 切换到 test 库
mysql> use test;
# 执行 explain 命令
mysql> explain select * from products where productName='geektime';
+----+-------------+----------+------+---------------+------+---------+------+-------+-------------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows  | Extra       |
+----+-------------+----------+------+---------------+------+---------+------+-------+-------------+
|  1 | SIMPLE      | products | ALL  | NULL          | NULL | NULL    | NULL | 10000 | Using where |
+----+-------------+----------+------+---------------+------+---------+------+-------+-------------+
1 row in set (0.00 sec)


```

/var/lib/mysql/test的4个文件：
* MYD 存储表的数据
* MYI 存储表的索引
* frm 存储表的元信息(表结构)
* opt 存储数据库的元信息(字符集)

没用索引....(top+strace)

另外, [opensnoop](https://github.com/brendangregg/perf-tools/blob/master/opensnoop)监控打开文件

## 案例分析: redis慢
写文件慢(top+strace)

# 系统I/O瓶颈分析套路

linux系统I/O栈
![](linux性能优化之磁盘文件-笔记/3.png)

![](linux性能优化之磁盘文件-笔记/4.png)

iostat 发现磁盘IO瓶颈->pidstat确认进程, 分析读写->strace/lsof分析应用程序

# I/O性能优化
## 应用程序优化
* 1.用追加写代替随机写, 减少寻址开销, 加快I/O写的速度
* 2.借助缓存I/O, 充分利用系统缓存, 降低实际I/O的次数(mysql的MyISAM,查询依赖缓存)
* 3.在应用程序内部构建自己的缓存, 控制缓存数据和生命周期(批价mdb_cache)
* 4.频繁读写同一块磁盘, mmap代替, 减少读写次数(直接I/O)
* 5.在需要同步写的场景中, 尽量将写请求合并, fsync()代替O_SYNC(mdb一次提交N条数据)
* 6.在多个应用程序共享磁盘时, 为保证I/O不被一个进程独占, 可以使用cgroups的I/O子系统来限制 
进程/进程组的IOPS和吞吐量
* 7.在使用CFQ调度器时, ionice调整进程I/O优先级, 提高核心应用的优先级

```
ionice -p 25089 -c 2 -n 7
使用 ionice 之前查一下帮助文件，-c 是指定调度类型，这里选择的是 2，best-effort； 
-n 指定调度优先级，0 最高，7最低；-p 是指定进程号

OPTIONS
-c The scheduling class. 1 for real time, 2 for best-effort, 3 for
idle....

```
## 文件系统优化
* 1.选择合适的文件系统, 如xfs比ext4支持更大的磁盘分区和文件数, 支持大于16T的磁盘
* 2.优化文件系统的配置, 文件系统特性/日志模式/挂载选项(tune2fs 查看文件系统超级块)
* 3.优化文件系统缓存, /proc/sys/vm/vfs_cache_pressure 优化内核回收目录项缓存和索引节点缓存
* 4.利用tmpfs内存文件系统, 获得更好的I/O性能

## 磁盘优化
* 1.SSD替代HDD
* 2.利用RAID磁盘阵列
* 3.选择合适的I/O调度算法, SSD和虚拟机的磁盘一般用noop算法, 数据库一般使用deadline算法
* 4.磁盘隔离, 为I/O压力重的应用配置单独磁盘, 如数据库,日志
* 5.在顺序读较多的场景中, 增大磁盘的预读数据配置
* 6.优化内核设备I/O选项, 例如增加队列长度, 提高吞吐(I/O延迟可能增大)

```
查看当前系统的I/O调度方法:
cat /sys/block/sda/queue/scheduler
noop anticipatory deadline [cfq]
```











 

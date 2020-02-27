---
title: linux性能优化之网络 笔记
date: 2019-03-05 10:23:34
tags:
categories: 读书笔记
---

# linux网络

## 网络模型

| OSI七层 | 功能 | 协议 |
|:-|:-|:-|
| 应用层 | 负责为程序提供统一的接口 | ftp telnet http dns|
| 表示层 | 负责把数据转换成兼容接收系统的格式 | NULL |
| 会话层 | 负责维护计算机之间的通信连接 | NULL |
| 传输层 | 负责为数据加上传输表头(包含port), 形成数据包 | TCP UDP |
| 网路层 | 负责数据的路由和转发 | IP ICMP IGMP RIP |
| 数据链路层 | 负责MAC寻址, 错误侦测和改错 | ARP RARP MTU PPP |
| 物理层 | 负责在物理网络中传输数据帧 | IEEE802 |

| TCP/IP四层 | 功能 |
|:-|:-|
| 应用层 | 负责向用户提供一组应用程序, 比如HTTP/DNS/FTP |
| 传输层 | 负责端到端的通信, 比如TCP/UDP |
| 网络层 | 负责网络包的封装、寻址和路哟, 比如IP/ICMP |
| 网络接口层 | 负责在物理网络中传输数据帧, 比如MAC寻址, 错误侦测和改错 |

<!-- more -->

## OSI和TCP/IP关系
![](linux性能优化之网络-笔记/1.png)

## linux网络栈
![](linux性能优化之网络-笔记/2.png)

__TCP头(最长60个字节)__
![](linux性能优化之网络-笔记/4.png)
__UDP头__
![](linux性能优化之网络-笔记/5.png)
__IP头__
![](linux性能优化之网络-笔记/6.png)


## Linux 通用 IP 网络栈的示意图
![](linux性能优化之网络-笔记/3.png)

## 性能指标

| 带宽 | 表示链路的最大传输,单位为b/s(比特/秒) |
|:-|:-|
|吞吐量 | 单位时间内成功传输的数据量b/s(比特/秒)或B/s(字节/秒), 网络使用率=吞吐量/带宽 |
|延时 | 网络请求发出后, 收到远端回复包的延迟 |
|PPS | packet per second(包/秒), 以网络包为单位的传输速率, 通常用来评估网络转发能力如硬件交换机 |

另外还有网络的可用性, 并发连接数, 丢包率, 重传率等指标

## 网络配置
```
[root@localhost ~]# ifconfig enp0s3
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.137.20  netmask 255.0.0.0  broadcast 192.255.255.255
        inet6 fe80::a00:27ff:fe41:f483  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:41:f4:83  txqueuelen 1000  (Ethernet)
        RX packets 1190  bytes 124844 (121.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1004  bytes 123892 (120.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
		
[root@localhost ~]# ip -s addr show dev enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:41:f4:83 brd ff:ff:ff:ff:ff:ff
    inet 192.168.137.20/8 brd 192.255.255.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe41:f483/64 scope link 
       valid_lft forever preferred_lft forever
    RX: bytes  packets  errors  dropped overrun mcast   
    137456     1311     0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    134854     1070     0       0       0       0    
```

* ifconfig中的RUNNING 和 ip中的LOWER_UP 表示网络物理层是连通的, 即网线插好了
* MTU 的大小
* 网络收发字节数、包数、错误数及丢包情况。详细指标如下

| errors | 发生错误的数据包数, 比如校验错误, 帧同步错误等 |
|:-|:-|
| dropped | 丢弃的数据包, 数据包已经收到了Ring Buffer, 但内存资源不足原因丢包 |
| overrun | 超限数据包数, 网络I/O速度过快, 导致Ring Buffer来不及处理(队列满) |
| carrier| 发送carrier错误的数据包数, 比如双工模式不匹配, 物理电缆线出问题等 |
| collsns| 碰撞数据包数 |


## 套接字信息
```
# head -n 3 表示只显示前面 3 行
# -l 表示只显示监听套接字
# -n 表示显示数字地址和端口 (而不是名字)
# -p 表示显示进程信息
$ netstat -nlp | head -n 3
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      840/systemd-resolve

# -l 表示只显示监听套接字
# -t 表示只显示 TCP 套接字
# -n 表示显示数字地址和端口 (而不是名字)
# -p 表示显示进程信息
$ ss -ltnp | head -n 3
State    Recv-Q    Send-Q        Local Address:Port        Peer Address:Port
LISTEN   0         128           127.0.0.53%lo:53               0.0.0.0:*        users:(("systemd-resolve",pid=840,fd=13))
LISTEN   0         128                 0.0.0.0:22               0.0.0.0:*        users:(("sshd",pid=1459,fd=3))

```

| QUEUE | LISTEN | ESTABLISHED | 
|:-|:-|:-|
| Recv-Q | (sync backlog当前值, 当前使用了多少)表示Accept queue中等待被服务器accept() | 套接字缓冲区中没有被应用收取的字节数(接受队列长度) | 
| Send-Q | (sync backlog最大值, 全连接队列长度)即为Accept queue的最大值 | 还没有被远端主机确认的字节数(发送队列长度) |

全连接的大小取决于min(backlog, somaxconn), backlog在创建socket时传入, somaxconn是系统参数
半连接的大小取决于max(64, /proc/sys/net/ipv4/tcp_max_syn_backlog), 不同版本会有差异

## 协议栈信息
```
[root@localhost ~]# netstat -s
...
Tcp:
    0 active connections openings
    1 passive connection openings
    0 failed connection attempts
    0 connection resets received
    1 connections established
    430 segments received
    266 segments send out
    0 segments retransmited
    0 bad segments received.
    0 resets sent
...
	
[root@localhost ~]# ss -s
Total: 187 (kernel 204)
TCP:   6 (estab 1, closed 0, orphaned 0, synrecv 0, timewait 0/0), ports 0

Transport Total     IP        IPv6
*         204       -         -        
RAW       0         0         0        
UDP       5         3         2        
TCP       6         4         2        
INET      11        7         4        
FRAG      0         0         0       
```

## 网络吞吐和PPS

```
// 1 表示每隔1秒输出一组数据
[root@localhost ~]# sar -n DEV 1
Linux 3.10.0-693.el7.x86_64 (localhost.localdomain)     03/05/2019      _x86_64_        (1 CPU)

10:06:56 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
10:06:57 PM    enp0s3      1.00      1.00      0.06      0.22      0.00      0.00      0.00
10:06:57 PM    enp0s8      0.00      0.00      0.00      0.00      0.00      0.00      0.00
10:06:57 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
10:06:57 PM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00
// rxpck/s 和 txpck/s 分别是接收和发送PPS, 单位为包/秒
// rxkB/s 和 txkB/s 分别是接收和发送吞吐量, 单位是KB/秒
// rxcmp/s 和 txcmp/s 分别是接收和发送的压缩包数据, 单位是包/秒
// %ifutil 网络接口使用率, max(rxkB/s, txkB/s)/Bandwidth

// 查看带宽, 如下为千兆网卡
[root@localhost ~]# ethtool enp0s8 |grep Speed
        Speed: 1000Mb/s
```

## 连通性和延时
```
[root@localhost ~]# ping www.baidu.com
PING www.a.shifen.com (183.232.231.172) 56(84) bytes of data.
64 bytes from 183.232.231.172 (183.232.231.172): icmp_seq=1 ttl=49 time=27.1 ms
64 bytes from 183.232.231.172 (183.232.231.172): icmp_seq=2 ttl=49 time=28.2 ms
64 bytes from 183.232.231.172 (183.232.231.172): icmp_seq=3 ttl=49 time=28.3 ms
64 bytes from 183.232.231.172 (183.232.231.172): icmp_seq=4 ttl=49 time=27.6 ms
64 bytes from 183.232.231.172 (183.232.231.172): icmp_seq=5 ttl=49 time=27.6 ms
^C
--- www.a.shifen.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 22064ms
rtt min/avg/max/mdev = 27.184/27.833/28.343/0.440 ms

// 第一部分是每个ICMP请求信息, 序列号, TTL(生存时间), 以及往返延时
// 第二部分是总的统计
```
-----
# C10K 和 C1000K
单机处理1W个并发连接请求

## I/O模型优化
I/O多路复用时, 两种事件通知方式
+ __水平触发__ 只要文件描述符可以非阻塞的执行I/O, 就会触发通知。应用程序可以随时检查fd的状态, 
然后根据状态进行I/O操作。
+ __边缘触发__ 只有在文件描述符状态发生改变时(I/O请求到达), 才会发生一次通知。此时需要应用 
尽可能多的执行I/O读写, 不及时处理通知就丢失了。

### 使用非阻塞I/O和水平触发通知(select/poll)
都需要对fds进行轮询; 在32位系统中, select描述符最多1024个; 并且, 在select内部检查套接字状态 
是用轮询, 再加上应用的轮询, 时间复杂度O(n^2).

poll对select改进, 使用固定长度的数组, 没有了最大描述符数量的限制; 应用程序还是需要轮询, 处理 
耗时跟描述符的数量就是O(n)关系.

另外应用程序调用select/poll, 需要把文件描述符集合从用户空间传入内核空间, 由内核空间修改后, 再 
传出到用户空间, 来回切换处理成本高.

### 使用非阻塞I/O和边缘触发通知(epoll)
+ epoll使用红黑树在内核管理描述符集合, 应用程序在每次操作时不需要传入传出这个集合
+ epoll使用时间驱动机制, 只关注事件发生的描述符(就绪链表), 不需要轮询整个集合

### 使用异步I/O
没有经验

## 工作模型优化
+ 主进程 + 多个worker子进程
+ 监听到相同端口多进程方式(开启SO_REUSEPORT), 只会有1个进程被唤醒

## C10M
跳过冗长的协议栈处理, 直接把网络包送到应用程序.有如下两种机制
+ DPDK 用户态网络标准, 跳过内核协议栈, 直接由用户态程序通过轮询来处理网络接收
+ XDP 允许网络包在进入内核前就进行处理(丢弃,转发,给上层应用)

![](linux性能优化之网络-笔记/7.png)
![](linux性能优化之网络-笔记/8.png)

-----
# 评估系统网络性能
各个协议层的基本测试

## 转发性能
pktgen

## TCP/UDP性能
```
[root@localhost ~]# yum install -y iperf3

# -s 表示启动服务端，-i 表示汇报间隔，-p 表示监听端口
[root@localhost ~]# iperf3 -s -i 1 -p 10000

# -c 表示启动客户端，192.168.0.30 为目标服务器的 IP
# -b 表示目标带宽 (单位是 bits/s)
# -t 表示测试时间
# -P 表示并发数，-p 表示目标服务器监听端口
[root@localhost ~]# iperf3 -c 127.0.0.1 -b 1G -t 15 -P 2 -p 10000
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-15.00  sec  1.74 GBytes   995 Mbits/sec    0             sender
[  4]   0.00-15.00  sec  1.74 GBytes   995 Mbits/sec                  receiver
[  6]   0.00-15.00  sec  1.74 GBytes   995 Mbits/sec    0             sender
[  6]   0.00-15.00  sec  1.74 GBytes   995 Mbits/sec                  receiver
[SUM]   0.00-15.00  sec  3.48 GBytes  1.99 Gbits/sec    0             sender
[SUM]   0.00-15.00  sec  3.48 GBytes  1.99 Gbits/sec                  receiver

```
从测试结果看带宽(吞吐量)超出预期1Gb/s

## HTTP性能
ab

## 应用负载性能
wk

## 总结
+ 应用层 用wrk,Jmeter
+ 传输层 iperf/netperf
+ 再向下 pktgen

-----
# DNS解析时快时慢

## 域名与DNS解析
DNS服务通过资源记录的方式, 管理数据
+ A记录 用来把域名转换成IP地址
+ CNAME记录 用来创建别名
+ NS记录 表示该域名对应的域名服务器地址

```
[root@localhost ~]# cat /etc/resolv.conf
; generated by /usr/sbin/dhclient-script
nameserver 10.70.75.253

[root@localhost ~]# yum install -y bind-utils

[root@localhost ~]# nslookup time.geekbang.org
Server:         10.70.75.253
Address:        10.70.75.253#53

Non-authoritative answer:
Name:   time.geekbang.org
Address: 39.106.233.176

[root@localhost ~]# dig +trace +nodnssec time.geekbang.org

; <<>> DiG 9.9.4-RedHat-9.9.4-73.el7_6 <<>> +trace +nodnssec time.geekbang.org
;; global options: +cmd
.                       518400  IN      NS      m.root-servers.net.
.                       518400  IN      NS      c.root-servers.net.
.                       518400  IN      NS      i.root-servers.net.
.                       518400  IN      NS      d.root-servers.net.
.                       518400  IN      NS      j.root-servers.net.
.                       518400  IN      NS      e.root-servers.net.
.                       518400  IN      NS      l.root-servers.net.
.                       518400  IN      NS      g.root-servers.net.
.                       518400  IN      NS      h.root-servers.net.
.                       518400  IN      NS      f.root-servers.net.
.                       518400  IN      NS      k.root-servers.net.
.                       518400  IN      NS      b.root-servers.net.
.                       518400  IN      NS      a.root-servers.net.
;; Received 239 bytes from 10.70.75.253#53(10.70.75.253) in 46 ms

org.                    172800  IN      NS      a0.org.afilias-nst.info.
org.                    172800  IN      NS      a2.org.afilias-nst.info.
org.                    172800  IN      NS      b0.org.afilias-nst.org.
org.                    172800  IN      NS      b2.org.afilias-nst.org.
org.                    172800  IN      NS      c0.org.afilias-nst.info.
org.                    172800  IN      NS      d0.org.afilias-nst.org.
;; Received 448 bytes from 199.7.91.13#53(d.root-servers.net) in 426 ms

geekbang.org.           86400   IN      NS      dns10.hichina.com.
geekbang.org.           86400   IN      NS      dns9.hichina.com.
;; Received 96 bytes from 199.19.56.1#53(a0.org.afilias-nst.info) in 106 ms

time.geekbang.org.      600     IN      A       39.106.233.176
;; Received 62 bytes from 106.11.211.56#53(dns10.hichina.com) in 40 ms

```

![](linux性能优化之网络-笔记/9.png)

可以把主机名和IP地址对应关系, 写入到本机/ect/hosts中, 指定的主机名可以直接找到IP
```
[root@localhost ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

[root@localhost ~]# ping localhost4
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.041 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.066 ms
^C
--- localhost ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.041/0.053/0.066/0.014 ms
```

## 案例

+ __DNS__ 未配置在/etc/resolve.conf 导致找到域名服务器
```
可以ping通IP, ping域名失败
# nslookup -debug time.geekbang.org
;; Connection to 127.0.0.1#53(127.0.0.1) for time.geekbang.org failed: connection refused.
;; Connection to ::1#53(::1) for time.geekbang.org failed: address not available.
```

+ DNS不稳定
```
# time nslookup time.geekbang.org
;; connection timed out; no servers could be reached
 
real	0m15.011s
user	0m0.006s
sys	0m0.006s

$ ping -c3 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=30 time=134.032 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=30 time=431.458 ms

延时太大导致

```

## 小结
DNS优化方法
+ 对DNS结果进行缓存
+ 对DNS解析的结果进行预取
+ 使用HTTPDNS代替常规的DNS解析, 避免域名劫持
+ 基于DNS的全局负载均衡(GSLB), 根据用户位置返回距离最近的IP

-----

# tcpdump和wireshark 分析网络流量

略过

-----

# 如何缓解DDos攻击带来的性能下降
DDoS 的前身是 DoS（Denail of Service），即拒绝服务攻击，指利用大量的 
合理请求，来占用过多的目标资源，从而使目标服务无法响应正常请求。
DDoS（Distributed Denial of Service） 则是在 DoS 的基础上，采用了分布 
式架构，利用多台主机同时攻击目标主机。

__攻击原理__
+ 耗尽带宽
+ 耗尽操作系统的资源
+ 消耗应用程序的运行资源


## 案例
__模拟__


__优化__
+ iptables 限制
+ 修改半连接数量限制
+ 开启 TCP SYN Cookies,  基于连接信息（包括源地址、源端口、目的地址、目的端口等）以及 
一个加密种子（如系统启动时间），计算出一个哈希值（SHA1），这个哈希值称为 cookie

## 小结
在 Linux 服务器中，你可以通过内核调优、DPDK、XDP 等多种方法，来增大服务器的抗攻击能力， 
降低 DDoS 对正常服务的影响。而在应用程序中，你可以利用各级缓存、 WAF、CDN 等方式， 
缓解 DDoS 对应用程序的影响。

-----

# 网络延迟变大

























---
title: linux高性能服务器编程
date: 2020-07-01 14:19:15
tags:高服务器编程
---







# 服务器编程

# TCP/IP协议详解

## TCP/IP协议簇

![image-20200810111010725](linux高性能服务器编程/image-20200810111010725.png)

数据链路层：实现了网卡接口的网络驱动程序，以处理数据在物理媒介(以太网)上的传输

网络层：数据包的选路和转发

传输层：为两台主机上的程序提供端到端服务

应用层：负责处理应用程序的逻辑



TCP协议：为应用层提供可靠的、面向连接的和基于流的服务

UDP协议：与TCP相反，为应用层提供不可靠、无连接和基于数据报的服务

ARP工作原理：主机向所在的网络广播一个ARP请求, 请求包含目标主机的网络地址。此网络上的其他主机收到这个请求，但只有被请求的目标机器会回应一个ARP应答，其中包含自己的物理地址。

```shell
# 查看ARP缓存
arp

# DNS服务文件
/etc/resolv.conf

# 查看DNS
host -t A www.baidu.com
```



## IP协议详解

![image-20200810112742000](linux高性能服务器编程/image-20200810112742000.png)

IP分片：

```shell
# MTU 1500, ICMP 头部8个字节, 所以1473 + 8 > 1500 , 可以分片
51_zjdev[/data01/zjgrp/zjdev]%ping 10.19.14.52 -s 1473
PING 10.19.14.52 (10.19.14.52) 1473(1501) bytes of data.
1481 bytes from 10.19.14.52: icmp_seq=1 ttl=64 time=1.08 ms
1481 bytes from 10.19.14.52: icmp_seq=2 ttl=64 time=0.494 ms
1481 bytes from 10.19.14.52: icmp_seq=3 ttl=64 time=0.553 ms

51_zjdev[/data01/zjgrp/zjdev]%sudo tcpdump -i eth0 -ntv icmp
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
IP (tos 0x0, ttl 64, id 541, offset 0, flags [+], proto ICMP (1), length 1500)
    10.19.14.51 > 10.19.14.52: ICMP echo request, id 16156, seq 1, length 1480
IP (tos 0x0, ttl 64, id 541, offset 1480, flags [none], proto ICMP (1), length 21)
    10.19.14.51 > 10.19.14.52: ip-proto-1

# 查看MTU
[root@localhost ~]# ifconfig -a
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.113  netmask 255.255.255.0  broadcast 192.168.1.255
```



IP路由机制：IP完全匹配，匹配网络段，默认路由

IP转发：/etc/sys/net/ipv4/ip_forward ，一般是接收和发送，但也可以转发

重定向：

IPv6头部结构：

![image-20200810141042471](linux高性能服务器编程/image-20200810141042471.png)

## TCP 协议详解

![image-20200810141250784](linux高性能服务器编程/image-20200810141250784.png)



TCP头部选项：

连接超时：iptables -I INPUT -p tcp -- syn -i eth0 -j DROP

TCP状态转移：

![image-20200810141623661](linux高性能服务器编程/image-20200810141623661.png)

TIME_WAIT存在原因：可靠的终止TCP链接；保证迟来的TCP报文有足够的时间被识别并被丢弃。

RST复位报文场景：访问不存在的端口，异常终止链接，处理半打开链接。

Nagel算法：在任意时刻，发送的TCP报文在未被确认到达之前不能发送其他报文，避免拥塞。

TCP超时重传：

TCP拥塞控制：慢启动，拥塞避免，快速重传，快速恢复





# 高性能服务器编程



## 网络编程基础API

### 网络地址

```c++
// 专用地址
#include <bits/socket.h>
struct sockaddr
{
    sa_family_t sa_family;
    char sa_data[14];
}

// 通用地址
struct sockaddr_in
{
    sa_family_t sin_family;  // AF_UNIX 地址簇
    u_uint16_t sin_port;     // 端口, 网络字节序
    struct in_addr sin_addr; // IPv4地址
}
struct in_addr
{
    u_uint32_t s_addr; // IPv4地址，网络字节序
}

// IPv6地址不一样

```

### 字节序转换

```c++
#incude <netinet.h>
unsigned long int htonl(unsigned long int hostlong);
unsigned short int htons(unsigned short int hostshort);

unsigned long int ntohl( unsigned long int netlong); 
unsigned short int ntohs( unsigned short int netshort);
```



### IP地址转换

```c++
#include <arpa/inet.h>
// 将十进制IP地址转换成网络字节序地址
int_addr_t inet_addr(const char* strptr);

// 同inet_addr, 返回结果在保存在inp中, 成功返回1，失败返回0
int inet_aton(const char* cp, struct in_addr* inp);

// 将网络地址转换成十进制点分字符串, 内部用静态变量存储结果
char* inet_ntoa(struct in_addr in);

// 通用转换, 支持IPv4和IPv6
#inlcude <arpa/inet.h>

// 转换成网络地址
int inet_pton(int af, const char* src, void* dst);
af: AF_UNIX/AF_UNIX6
src: 十进制网络地址
dst: 网络字节序地址

// 转换成十进制地址
const char* inet_ntop(int af, const void* src, char* dst, socklen_t cnt);
cnt: 存储单元大小，可用宏定义

#define INET_ADDRSTRLEN 16
#define INET6_ADDRSTRLEN 46
```



### 创建socket

```c++
#include <sys/type.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
domain: PF_INET/PF_INET6
type: SOCK_STREAM/SOCK_DGRAM   // &SOCK_NONBLOCK or &SOCK_CLOEXEC
return: fd 成功, -1 失败
```



### 命名socket

```c++
#include <sys/type.h>
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr* my_addr, socklen_t addrlen);
return: 0 成功, 1 失败
```



### 监听socket

```c++
#include <sys/type.h>
#incude <sys/socket.h>

int listen(int socketfd, int backlog);
backlog: 半连接（SYN_RCVD）队列大小
return: 0 成功 -1 失败
```



### 接受连接

```c++
#include <sys/type.h>
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr* addr, socklen_t* addrlen );
sockfd: 监听fd
addr: 对端地址
addrlen: 地址长度
return: fd 成功，-1 失败
desc: 从监听队列(半连接队列)中取出fd
```



### 发起连接

```c++
#include <sys/type.h>
#include <sys/socket.h>

int connect(int sockfd, struct sockaddr* serv_addr, socklen_t len);
sockfd: socket调用返回套接字
serv_addr: 服务端地址
socklen_t: 服务端长度
return: 0 成功, -1 失败
errno: ECONNREFUSED 目标端口不存在，被拒绝
	   ETIMEDOUT  连接超时
```



### 关闭连接

```c++
#include <unistd.h>

int close(int fd);

desc: 将fd引用计数-1, 如fork时fd引用计数+1, 为0时关闭连接
      读写同时关闭

#include <sys/socket.h>
    
int shutdown(int sockfd, int howto);
howto: SHUT_RD/ SHUT_WR/ SHUT_RDWR
desc: 可以将读写分别单独关闭
return: 0 成功, -1 失败

```



### 读写数据

```c++
// ---------------------
// TCP数据读写专用函数-----
#include <sys/type.h>
#include <sys/socket.h>

int recv(int sockfd, void* buf, size_t len, int flags);
return: 0 对端已关闭,  -1 失败
flags: 见截图 MSG_OOB 外带数据(紧急指针)一般让对端丢弃缓冲区数据，如ftp中断命令
另外要接收SIGURG信号, 需设置套接字归进程所有,带外数据不论发送多少只能读取一个有效字节
    
int send(int sockfd, const void* buf, size_t len, int flags);
len: 指定读写大小
return: -1 失败
    
// ---------------------
// UDP数据读写专用函数-----
#inlucde <sys/type.h>
#include <sys/socket.h>
    
int recvfrom(int sockfd, void* buf, size_t len, int flags, struct sockaddr* src_addr, socklen_t* addrlen);

int sendto(int sockfd, const void* buf, size_t len, int flags, struct sockaddr* dst_addr, socklen_t addrlen);

desc: 设置地址为NULL， 可以用于TCP

// ---------------------
// 数据读写通用函数--------
#inlcude <sys/socket.h>

int recvmsg(int sockfd, struct msghdr *msg, int flags);

int sendmsg(int sockfd, struct msghdr *msg, int flags);
    
struct msghdr { 
    void* msg_name;/* socket 地址*/ 
    socklen_t msg_namelen;/* socket 地址 的 长度*/ 
    struct iovec* msg_iov;/* 分散 的 内存 块， 见 后文*/ 
    int msg_iovlen;/* 分散 内存 块 的 数量*/ 
    void* msg_control;/* 指向 辅助 数据 的 起始 位置*/ 
    socklen_t msg_ controllen;/* 辅助 数据 的 大小*/ 
    int msg_flags;/* 复制 函数 中的 flags 参数， 并在 调用 过程中 更新*/ 
};

struct iovec { 
    void* iov_base;/* 内存 起始 地址*/ 
    size_ t iov_len;/* 这块 内存 的 长度*/ 
};

```

![image-20200702100417447](linux高性能服务器编程/image-20200702100417447.png)

### 带外标记

```c++
#include <sys/socket.h>

int sockatmark(int sockfd);
return: 0 无外带数据, 1 有外带数据(recv, MSG_OOB接收紧急数据)
```



### 从fd获取地址信息

```c++
#include <sys/socket.h>

int getsockname(int sockfd, struct sockaddr* addr, socklen_t* len);
int getpeername(int sockfd, struct sockaddr* addr, socklen_t* len);

return: 0 sucess, -1 failed
desc: 结果存储在addr变量中
```



### socket 选项

```c++
#include <sys/socket.h>

int getsockopt(int sockfd, int level, int option_name, void* option_value, socklen_t* len);

int setsockopt(int sockfd, int level, int option_name, void* option_value, socklen_t len);
/*
level: SOL_SOCKET 通用sock/ IPPROTO_IP / IPPROTO_IP6 / IPPROTO_TCP

option_name: SO_REUSEADDR  可占用TIME_WAIT状态的socket
             SO_RCVBUF, SO_SNDBUF 缓冲区大小
    		 SO_RCVLOWAT, SO_SNDLOWAT 可读写低水位标志, 默认1字节, 可读/可写 大于 低水位，通知程序可读/可写
    		 SO_LINGER 控制close关闭TCP连接的行为
*/

struct linger { 
    int l_ onoff; // 开启（ 非 0） 还是 关闭（ 0） 该 选项
    int l_ linger; // 滞留 时间
};
```



### 网络信息API

```c++
#include <netdb.h>

// ------------------
// 获取服务完整信息-----
struct hostent* gethostbyname(const char* name);
struct hostent* gethostbyaddr(const void* addr, size_t len, int type);
addr: 目标主机IP地址
len: 地址长度
type: AF_INET/AF_INET6
desc: 获取主机完整信息
    
struct hostent { 
    char* h_name;/* 主机 名*/ 
    char** h_aliases;/* 主机 别名 列表， 可能有 多个*/ 
    int h_addrtype;/* 地址 类型（ 地址 族）*/ 
    int h_length;/* 地址 长度*/ 
    char** h_addr_list/* 按 网络 字节 序 列出 的 主机 IP 地址 列表*/ 
};

// ------------------
// 获取服务完整信息-----
struct servent* getservbyname(const char* name, const char* proto);
struct servent* getservbyport(int port, const char* proto);
proto: "TCP"/ "UDP"
    
struct servent { 
    char* s_name;/* 服务 名称*/ 
    char** s_aliases;/* 服务 的 别名 列表， 可能有 多个*/ 
    int s_port;/* 端口 号*/ 
    char* s_proto;/* 服务 类型, 通常 是 tcp 或者 udp*/ 
}; 
 
// ------------------
// 通过主机名获取IP地址，通过服务名获取端口号
int getaddrinfo(...);
void freeaddrinfo(struct addrinfo* res);

// 通过socket 地址获取主机名或服务名
int getnameinfo(...);
```

### 错误码转换

```c++
#include <netdb.h>

const char* gai_strerror(int error);
```



## 高级I/O函数

### pipe

```c++
#include <unistd.h>

int pipe(int fd[2]);
fd[0] 和 fd[1] 分别对应管道的两端, fd[0]写入只能从fd[1]读出

#include <sys/type.h>
#include <sys/socket.h>
    
int socketpair(int domain, int type, int fd[2]);
domain: AF_UNIX
desc: 双向管道
```



### dup/dup2

```c++
#include <unistd.h>

int dup(int file_describe);
int dup2(int file_fd1, int file_fd2);
return: 系统当前可用最小整数值
desc: 把标准输入重定向到文件 或标准输出重新定到文件，管道，网络

close(STDOUT_FILENO); // 关闭标准输出, 系统值1
dup(sockfd); // 返回1, 重定向到了网络

```



### readv/writev

```c++
#include <sys/uio.h>

ssize_t readv( int fd, const struct iovec* vector, int count);
ssize_t writev( int fd, const struct iovec* vector, int count);

// 简化版recvmsg,sendmsg
```



### sendfile

```c++
#include <sys/sendfile.h>

ssize_t sendfile(int out_fd, int in_fd, off_t* offset, size_t count);
out_fd: 待写入fd
in_fd: 待读出fd
offset: 开始位置
count: 长度
// 描述符之间直接传递数据，在内核中完成，零拷贝
```



### mmap/munmap

```c++
#include <sys/mman.h>

// 申请一段内存空间, 进程间通信的共享内存
void* mmap(void* start, size_t length, int prot, int flags, int fd, off_t offset);
start: 指定起始地址, NULL时系统自动分配
prot: 权限 PROT_READ/PORT_WIRTE/PROT_EXEC/PROT_NONE
flags: 控制内容被修改后程序行为 MAP_SHARED
fd: 文件描述符
offset: 文件映射便宜位置,不需要映射整个文件
return:　返回指向内存的指针

int munmap(void* start, size_t length);


```



### splice

```c++
#include <fcntl.h>

// 用于两个文件描述符之间移动数据，也是零拷贝
ssize_ t splice( int fd_in, loff_t* off_in, int fd_out, loff_t* off_out, size_t len, unsigned int flags);
flags: SPLICE_F_NONBLOCK

```

发

### tee

```c++
#include <fcntl.h>

// 管道文件描述符之间复制数据
ssize_t tee( int fd_in, int fd_out, size_t len, unsigned int flags);

```



### fcntl

```c++
#include <fcntl.h>

int fcntl(int fd, int cmd...);

cmd: F_GETFL/F_SETFL

int setnonblocking( int fd) 
{ 
	int old_ option= fcntl( fd, F_GETFL);
	int new_ option= old_option| O_NONBLOCK;
	fcntl( fd, F_SETFL, new_ option); 
	return old_ option;
}
```

## linux 服务程序规范

### 日志

```c++
#include <syslog.h>

void syslog(int priority, const char* message...);

// 设置格式
void openlog(const char* ident, int logopt, int facility);
ident: 指定字符串添加到日期和时间之后
logopt: LOG_PID 日志消息中包含进行ID
    
// 设置日志级别
int setsyslogmask(int mask);

// 关闭日志功能
void closelog();
```



### 用户信息

```c++
#include <sys/type.h>
#include <unistd.h>

uid_t getuid();
uid_t geteuid();
gid_t getgid();
gid_t getegid();

...set...

// 有效用户ID, 任何用户执行su命令，有效用户就是root，可以访问/etc/passwd
// uid 是启动进程的用户ID, euid是文件所有者的ID
```



### 进程间关系

```c++
#include <unistd.h>

// 进程组ID
pid_t getpgid(); 
int setpgid(pid_t pid, pid_t pgid);

// 会话 = 进程组首领的PGID
pid_t getsid(pid_t pid);
pid_t setsid(void);

// example
ps -o pid ,ppid, gpid, sid, comm |less
```



### 系统资源限制

```c++
#include <sys/resource.h>

int getrlimmt(int resource, struct rlimit* rlim);
int setrlimit(int resource, struct rlimt* rlim);
```



### 改变工作目录和根目录

```c++
#include <unistd.h>

char* getcwd(char* buf, size_t size);
buf: 存放工作目录绝对路径的存储
size: 存储长度

int chdir(const char* path);
path: 切换的目标目录

int chroot(const char* path);
path: 目标根目录
desc: 读取的是新根下的目录和文件，这是一个与原系统根下文件不相关的目录结构
```



### 服务程序后台化

```c++
#include <>

int daemon(int nochdir, int noclose);
nochdir: 0 表示"/", 否则用当前工作目录
noclose: 0 标准输入/输出/错误 重定向到 /dev/null
```



## 高性能服务器框架

### 服务器模型

异步I/O 立即返回，不论是I/O否阻塞，内核完成I/O操作后通知应用程序。
同步I/O 向应用程序通知的是就绪事件(I/O阻塞，复用，信号)，而异步I/O通知的是完成事件。

服务器一般处理三种事件：I/O事件，信号事件，定时时间。

![image-20200811095227925](linux高性能服务器编程/image-20200811095227925.png)

### 两种事件处理模式

Reactor模式：主线程监听，可读/可写 事件 放入请求队列，唤醒工作线程处理
Proactor模式：主线程处理所有I/O操作，通知工作线程读写结果



### 两种并发编程模式

如果程序是计算密集型，并发编程没有优势。如果程序是I/O密集型，等待I/O线程放弃CPU，其他线程执行。

__半同步/半异步__：同步是指代码按照顺序执行，异步是指程序执行需要系统事件来驱动。
同步线程处理逻辑，异步线程处理I/O事件。
异步线程监听请求，派发socket，通知某个工作在同步模式的线程读取并处理。如下图中每个线程都有自己的监听。

![image-20200811100609440](linux高性能服务器编程/image-20200811100609440.png)





__领导者/追随者__：多个工作线程轮流，在某一时间点，只有一个领导者，负责监听I/O事件，其他线程则都是追随者，休眠在线程池中等待成为领导者。缺点是一个线程无法管理多个连接。

组件包括：句柄集(HandleSet)，线程集(ThreadSet)，事件处理器(EventHandle)，具体事件处理器(ConcreteEventHanle)

![image-20200811101730144](linux高性能服务器编程/image-20200811101730144.png)







### 有限状态机

逻辑单元内部高效编程方法，一个消息码对应一个逻辑函数。

### 其他高效编程建议

池、数据复制、上下文切换和锁



## I/O 复用

### select

```c++
#include <sys/select.h>
int select(int nfds, fd_set* readfds, fd_set* writefds, fd_set*exceptfds, timeval* timeout);

// nfds 最大描述符的值+1
// 返回0 没有描述符就绪; 返回-1 失败; 收到信号 立即返回-1 errno EAGAIN

// 设置fd_set
FD_ZERO( fd_ set* fdset);/* 清除 fdset 的 所有 位*/ 
FD_SET( int fd, fd_ set* fdset);/* 设置 fdset 的 位 fd*/ 
FD_CLR( int fd, fd_ set* fdset);/* 清除 fdset 的 位 fd*/ 
int FD_ISSET( int fd, fd_ set* fdset); /* 测试 fdset 的 位 fd 是否 被 设置*/

// 0 立即返回， NULL 一直阻塞直到有描述符就绪
struct timeval { 
long tv_ sec;	/* 秒数*/ 
long tv_ usec;	/* 微秒数*/
}
```

__文件描述符就绪条件__

可读：

> 1 内核读缓冲区可读数据大于或等于其低水位标记SO_RCVLOWAT
> 2 对端关闭连接，该socket的读返回0
> 3 监听socket上有新的连接请求
> 4 socket上有未处理的错误。用getsockopt来读取和清除该错误

可写：

> 1 内核写缓冲区空闲数据大于或等于其低水位标记SO_SNDLOWAT
> 2 socket写操作关闭，对该被关闭的socket执行写操作会触发一个SIGPIPE信号
> 3 socket使用非阻塞connect 连接成功或失败(超时)之后  ，其中errno=EINPROGRESS 正在建立连接
> 4 socket上有未处理的错误。用getsockopt来读取和清除该错误

```c++

int error= 0; 
socklen_ t length= sizeof( error); 
/*调用 getsockopt 来 获取 并 清除 sockfd 上 的 错误*/ 
if( getsockopt( sockfd, SOL_SOCKET, SO_ERROR, &error, &length) ＜ 0)
{
}
```



异常：

> 外带数据



### poll

```c++
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

// fds 是一个pollfd类型的数组
struct pollfd
{
    int fd;	/* 文件描述符*/
    short events; /* 注册事件*/
    short revents; /* 就绪事件，由内核填充*/
}
// nfds 个数
// timeout -1 一直阻塞，0 立即返回
// 返回值同select
```



### epoll

```c++
#include <sys/epoll.h>
int epoll_create(int size);
// 创建内核事件表

int epoll_ctl(int epfd, in op, int fd, epoll_event* event);
// 操作内核事件表 
// op = EPOLL_CTL_ADD, EPOLL_CTL_DEL, EPOLL_CTL_MOD
struct epoll_event
{
    uint32_t events; /* 事件 EPOLLIN EPOLLOUT EPOLLET EPOLLONESHOT*/
    epoll_data_t data; /* 用户数据 常用的是data.fd */
}


int epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout);
// 等待就绪, events 是数组
```



LT 水平触发，默认方式，事件不处理下次还会触发；

ET 边缘触发，事件必须被处理，下次不会触发。



### 区别比较

|                                  | select                                                       | poll                                                         | epoll                                                        |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 事件集合                         | 3个参数分别传入感兴趣的可读、可写、异常等事件，内核修改反馈其中就绪事件。每次调用都需要重置这3个参数 | 统一处理所有时间类型，因此只需要一个参数。用户通过pollfd.events传入感兴趣事件，内核通过修改pollfd.revents反馈其中就绪的事件 | 内核通过一个事件表管理用户感兴趣的事件。因此epoll_wait时无需反复传入用户感兴趣的事件。epoll_wait第二个参数events仅用来反馈就绪的事件 |
| 应用索引就绪文件描述符时间复杂度 | O(n)                                                         | O(n)                                                         | O(1)                                                         |
| 支持最大文件描述符数量           | 一般限制是1024                                               | 65535                                                        | 65535                                                        |
| 工作模式                         | LT                                                           | LT                                                           | LT/ET                                                        |
| 内核实现和工作效率               | 采用轮询方式来检测就绪事件，O(n)                             | 轮询，O(n)                                                   | 采用回调方式来检测就绪事件，O(1)                             |



### 同时处理TCP和UDP

tcpfd和udpfd可以同时绑定到一个端口，把两个fd都加到事件列表中可以同时监听



## 信号





## 定时器




















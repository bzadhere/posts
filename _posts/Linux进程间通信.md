---
title: Linux进程间通信
date: 2018-10-24 21:19:07
tags:
categories: linux
---

------

>  摘自 IBM developerworks 郑彦兴

## [深刻理解Linux进程间通信(IPC)](https://www.ibm.com/developerworks/cn/linux/l-ipc/index.html)

<!-- more -->

## [Linux环境进程间通信（一）](https://www.ibm.com/developerworks/cn/linux/l-ipc/part1/index.html)

## [Linux环境进程间通信（二）信号（上）](https://www.ibm.com/developerworks/cn/linux/l-ipc/part2/index1.html)

## [Linux环境进程间通信（二）信号（下）](https://www.ibm.com/developerworks/cn/linux/l-ipc/part2/index2.html)

## [Linux环境进程间通信（三）消息队列](https://www.ibm.com/developerworks/cn/linux/l-ipc/part3/)

## [Linux环境进程间通信（四）信号灯](https://www.ibm.com/developerworks/cn/linux/l-ipc/part4/index.html)

## [Linux环境进程间通信（五）共享内存（上）](https://www.ibm.com/developerworks/cn/linux/l-ipc/part5/index1.html)

## [Linux环境进程间通信（五）共享内存（下）](https://www.ibm.com/developerworks/cn/linux/l-ipc/part5/index2.html)

## [Linux 环境进程间通信（六）套接口](https://www.ibm.com/developerworks/cn/linux/l-ipc/part6/index.html)

## 小结
* 管道及有名管道: 具有亲缘关系的进程间通信, 有名管道克服了管道没有名字的限制, 可用于无亲缘关系进程间通信
* 信号: 通知某进程某个事件发生, 可发送给自身。
* 报文队列(消息队列): 消息队列是消息的链接, 随内核持续; 克服了信号承载信息少, 管道只能承载无格式字节流以及缓冲区大小受限制缺点
* 信号量(semaphore): 进程间及同一进程不同线程间的同步手段。
* 共享内存: 使多个进程可以访问同一块内存, 是最快的可用IPC形式。针对其他通信机制运行效率低设计。可与信号量配合使用, 
达到进程间的同步和互斥
* 套接口(socket): 用于不同机器之间的进程间通信。

## 管道及有名管道
```
#include <unistd.h>
int pipe(int fd[2])

#include <sys/types.h>
#include <sys/stat.h>
int mkfifo(const char * pathname, mode_t mode)
```
写端依赖读端, 读端关闭, SIGPIPE信号默认停止

## 信号
__信号本质__
信号是对软中断机制的模拟, 信号是异步的

__信号种类__
可靠信号和不可靠信号, 可靠信号支持排队不会丢失(信号值位于SIGRTMIN及SIGRTMAX之间);
实时信号和非实时信号, 实时信号都是可靠信号, 支持排队, 非实时信号都是不可靠信号

信号安装sigaction比signal 多了一个信息传递

__进程对信号响应__
忽略/捕捉/默认缺省操作, 其中SIGSTOP/SIGKILL不能忽略

__信号发送__
发送信号的主要函数有：kill()、raise()、 sigqueue()、alarm()、setitimer()以及abort()。

__信号安装__
```
#include <signal.h> 
typedef void (*sighandler_t)(int)； 
sighandler_t signal(int signum, sighandler_t handler)); 

#include <signal.h> 
int sigaction(int signum,const struct sigaction *act,struct sigaction *oldact));
```

__信号阻塞和信号未决__
```
#include <signal.h>
int  sigprocmask(int  how,  const  sigset_t *set, sigset_t *oldset))；
int sigpending(sigset_t *set));
int sigsuspend(const sigset_t *mask))；
```

__信号生命周期__
信号的诞生->信号在进程中注册->信号在进程中注销->信号处理函数执行完毕

在进程的task_struct中有信号链表, 注册就是将信号加入到链表中。可靠信号 
一定会加入, 不可靠信号如有同一信号注册, 只会注册一次。

__信号编程注意事项__

为增强程序稳定性, 信号处理程序中应当使用可重入函数, 避免信号处理执行函数 
执行后修改数据导致出现不可预料结果。

满足下列要求的基本是不可再入:
+ 使用静态数据结构, 如getlogin()，gmtime()，getgrgid()，getgrnam()，getpwuid()以及getpwnam()等等
+ 函数实现时，调用了malloc() 或者free()函数
+ 实现使用了标准I/O函数， 如printf是可不再入

read/write/_exit/open 等是可再入函数

## 消息队列
__基本概念__
消息队列是随内核持续的一个消息链表(struct ipc_ids msg_ids)

__消息队列操作__
```
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

key_t ftok (char*pathname, char proj)；

// 不够直观
int ipc(unsigned int call, int first, int second, int third, void * ptr, long fifth);

// 获取消息队列描述符
int msgget(key_t key, int msgflg)
// 读取一个消息, 存储在msgbuf中
int msgrcv(int msqid, struct msgbuf *msgp, int msgsz, long msgtyp, int msgflg);
// 向队列发送一个消息, 发送消息+队列中消息>=容量时会阻塞
int msgsnd(int msqid, struct msgbuf *msgp, int msgsz, int msgflg);
// 设置属性或删除消息队列
int msgctl(int msqid, int cmd, struct msqid_ds *buf);

```

## 信号量

__概念__
随内核持续, 进程间的同步手段(struct ipc_ids sem_ids)


__操作__
```
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

// 文件名到键值映射
key_t ftok (char*pathname, char proj)

// key 键值, nsems 信号量个数, 
int semget(key_t key, int nsems, int semflg)

// sops指向数组的每一个sembuf结构, nsops为sops指向数组的大小
int semop(int semid, struct sembuf *sops, unsigned nsops)

int semctl(int semid，int semnum，int cmd，union semun arg)
```

## 共享内存

```
// 建立和解除映射关系
void* mmap ( void * addr , size_t len , int prot , int flags , int fd , off_t offset ) 
int munmap( void * addr, size_t len ) 

#include <sys/ipc.h>
#include <sys/shm.h>
API：shmget()、shmat()、shmdt()及shmctl()
```
区别是随进程和随内核

## 套接口
```
// 套接口是由socket数据结构代表的
struct socket
{
socket_state  state;     /* 指明套接口的连接状态，一个套接口的连接状态可以有以下几种
套接口是空闲的，还没有进行相应的端口及地址的绑定；还没有连接；正在连接中；已经连接；正在解除连接。 */
  unsigned long    flags;
  struct proto_ops  ops;  /* 指明可对套接口进行的各种操作 */
  struct inode    inode;    /* 指向sockfs文件系统中的相应inode */
  struct fasync_struct  *fasync_list;  /* Asynchronous wake up list  */
  struct file    *file;          /* 指向sockfs文件系统中的相应文件  */
struct sock    sk;  /* 任何协议族都有其特定的套接口特性，该域就指向特定协议族的套接口对
象。 */
  wait_queue_head_t  wait;
  short      type;
  unsigned char    passcred;
};

// 描述套接口通用地址的数据结构struct sockaddr
// 由于历史的缘故，在bind、connect等系统调用中, 地址都需要强转
struct sockaddr {
    sa_family_t sa_family;  /* address family, AF_xxx   */
    char        sa_data[14];    /* 14 bytes of protocol address */
};

// 描述因特网地址结构的数据结构struct sockaddr_in
struct sockaddr_in
  {
    __SOCKADDR_COMMON (sin_);   /* 描述协议族 */
    in_port_t sin_port;         /* 端口号 */
    struct in_addr sin_addr;        /* 因特网地址 */
    /* Pad to size of `struct sockaddr'.  */
    unsigned char sin_zero[sizeof (struct sockaddr) -
               __SOCKADDR_COMMON_SIZE -
               sizeof (in_port_t) -
               sizeof (struct in_addr)];
  };

// 接口
int socket( int domain, int type, int ptotocol)
int bind( int sockfd, const struct sockaddr * my_addr, socklen_t my_addr_len)
int connect( int sockfd, const struct sockaddr * servaddr, socklen_t addrlen)
int accept( int sockfd, struct sockaddr * cliaddr, socklen_t * addrlen)
```

## 补充

```
ipcs用法 
ipcs -a  是默认的输出信息 打印出当前系统中所有的进程间通信方式的信息
ipcs -m  打印出使用共享内存进行进程间通信的信息
ipcs -q   打印出使用消息队列进行进程间通信的信息
ipcs -s  打印出使用信号进行进程间通信的信息

ipcrm用法 
ipcrm -M shmkey  移除用shmkey创建的共享内存段
ipcrm -m shmid    移除用shmid标识的共享内存段
ipcrm -Q msgkey  移除用msqkey创建的消息队列
ipcrm -q msqid  移除用msqid标识的消息队列
ipcrm -S semkey  移除用semkey创建的信号
ipcrm -s semid  移除用semid标识的信号


```






















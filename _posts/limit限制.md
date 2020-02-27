---
title: limit限制
date: 2019-01-17 15:56:10
tags:
categories: linux
---

# 系统调用ulimit
```
[root@zhangbb ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 3893
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65535
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 65535
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

# 库函数
> #include <sys/time.h>  
> #include <sys/resource.h>  
> int getrlimit(int resource, struct rlimit *rlim);  
> int setrlimit(int resource, const struct rlimit *rlim);

```
#include <sys/resource.h>
#include <stdio.h>
#include <iostream>

int main()
{
        struct rlimit r;
    if(getrlimit(RLIMIT_AS, &r) < 0)
    {
        perror("getrlimit error");
        return 0;
    }
    printf("RLIMIT_AS cur %d, max %d\n", r.rlim_cur, r.rlim_max);

    if(getrlimit(RLIMIT_STACK, &r) < 0)
    {
        perror("getrlimit error");
        return 0;
    }
    printf("RLIMIT_STACK cur %d, max %d\n", r.rlim_cur, r.rlim_max);

    if(getrlimit(RLIMIT_MEMLOCK , &r) < 0)
    {
        perror("getrlimit error");
        return 0;
    }
    printf("RLIMIT_MEMLOCK cur %d, max %d\n", r.rlim_cur, r.rlim_max);
        return 0;
}

52_zjdev[/data01/zjgrp/zjdev/users/zhangbb]%./test
RLIMIT_AS cur -1, max -1
RLIMIT_STACK cur -1, max -1
RLIMIT_MEMLOCK cur 65536, max 65536
```

[参考](https://blog.csdn.net/Royecode/article/details/48738681)
# getrusage
获取运行进程使用资源

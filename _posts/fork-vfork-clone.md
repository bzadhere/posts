---
title: fork/vfork/clone
date: 2018-11-20 15:53:20
tags:
categories: linux
---

# fork/vfork/clone
| 系统调用 | 描述 |
|:-|:- |
| fork | fork创造的子进程是父进程的完整副本，复制了父亲进程的资源，包括内存的内容task_struct内容 |
| vfork | vfork创建的子进程与父进程共享数据段,而且由vfork()创建的子进程将先于父进程运行 |
| clone | Linux上创建线程一般使用的是pthread库 实际上linux也给我们提供了创建线程的系统调用，就是clone |

<!-- more -->
# fork
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main(void)
{
    int count = 1;
    int child;

    child = fork( );

    if(child < 0)
    {
        perror("fork error : ");
    }
    else if(child == 0)     //  fork return 0 in the child process because child can get hid PID by getpid( )
    {
        printf("This is son, his count is: %d (%p). and his pid is: %d\n", ++count, &count, getpid());
    }
    else                    //  the PID of the child process is returned in the parent’s thread of execution
    {
        printf("This is father, his count is: %d (%p), his pid is: %d\n", count, &count, getpid());
    }

    return EXIT_SUCCESS;
}

output:
This is father, his count is: 1 (0x7ffe32f99cc8), his pid is: 26649
This is son, his count is: 2 (0x7ffe32f99cc8). and his pid is: 26650

```
父子两个进程的pid不同，堆栈和数据资源都是完全的复制
子进程改变了count的值，而父进程中的count没有被改变
子进程与父进程count的地址（虚拟地址）是相同的（注意他们在内核中被映射的物理地址不同）

## 写入时复制(Copy-on-write)
完全复制包含4个步骤, 非常耗时
> 为子进程的页表分配页帧
> 为子进程的页分配页帧
> 初始化子进程的页表
> 把父进程的页复制到子进程相应的页中

写时复制则父进程和子进程共享页帧而不是复制页帧, 页面的访问权限也设成只读, 当发生修改时 
产生页面出错异常(page_fault int14)中断,此时CPU会执行系统提供的异常处理函数do_wp_page()  
来解决这个异常. do_wp_page()会对这块导致写入异常中断的物理页面进行取消共享操作,为写进  
程复制一新的物理页面, 使父进程A和子进程B各自拥有一块内容相同的物理页面.

# vfork
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main(void)
{
    int count = 1;
    int child;

   // child = vfork( );

    printf("Before create son, the father's count is:%d\n", count);

    if((child = vfork())< 0)
    {
        perror("fork error : ");
    }
    else if(child == 0)     //  fork return 0 in the child process because child can get hid PID by getpid( )
    {
        printf("This is son, his count is: %d (%p). and his pid is: %d\n", ++count, &count, getpid());
        exit(0);
    }
    else                    //  the PID of the child process is returned in the parent’s thread of execution
    {
        printf("After son, This is father, his count is: %d (%p), his pid is: %d\n", ++count, &count, getpid());
        exit(0);
    }

    return EXIT_SUCCESS;
}

output:
Before create son, the father's count is:1
This is son, his count is: 2 (0x7ffe635dbc18). and his pid is: 26921
After son, This is father, his count is: 3 (0x7ffe635dbc18), his pid is: 26920
```

vfork创建出的子进程（线程）共享了父进程的count变量，2者的count指向了同一个内存

注意事项：
> 由vfork创造出来的子进程还会导致父进程挂起，除非子进程exit或者execve才会唤起父进程
> 由vfok创建出来的子进程共享了父进程的所有内存，包括栈地址，直至子进程使用execve启动新的应用程序为止
> 由vfork创建出来得子进程不应该使用return返回调用者，或者使用exit()退出，但是它可以使用_exit()函数来退出

## fork/vfork 区别联系
| func | copy | run |
|:-|:-|:-|
| fork| 子进程拷贝父进程的数据段，代码段| 执行次序不确定 |
| vfork| 子进程与父进程共享数据段 | 保证子进程先运行 |

# clone
```
int clone(int (fn)(void ), void *child_stack, int flags, void *arg); 

#include <stdio.h>
#include <malloc.h>
#include <stdlib.h>

#include <sched.h>
#include <signal.h>

#include <sys/types.h>
#include <unistd.h>


#define FIBER_STACK 8192
int a;
void * stack;

int do_something(void*)
{
    printf("This is son, the pid is:%d, the a is: %d\n", getpid(), ++a);
    free(stack); 
    exit(1);
}

int main()
{
    void * stack;
    a = 1;
    stack = malloc(FIBER_STACK);//为子进程申请系统堆栈

    if(!stack)
    {
        printf("The stack failed\n");
        exit(0);
    }
    printf("creating son thread!!!\n");

    clone(&do_something, (char *)stack + FIBER_STACK, CLONE_VM|CLONE_VFORK, 0);//创建子线程

    printf("This is father, my pid is: %d, the a is: %d\n", getpid(), a);
    exit(1);
}

output:
creating son thread!!!
This is son, the pid is:27528, the a is: 2
This is father, my pid is: 27527, the a is: 2
```

## clone/fork/vfork区别与联系
最终都是调用do_fork函数完成, 只是参数不同

















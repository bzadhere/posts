---
title: gdb多线程调试
date: 2018-10-15 14:48:04
tags: gdb
categories: gdb
---

> * all-stop mode: 默认模式, 有一个线程在断点处停止，其他所有线程也会停止
> * non-stop mode: 某一个线程停止时，其他线程会继续运行
> * Background Execution：异步运行程序
> * Thread-Specific Breakpoints: 控制断点
> * Interrupted System Calls: gdb会干扰系统调用
> * Observer Mode：gdb不影响程序执行

<!-- more -->

- [x] all-stop mode
当进程在gdb下停止时，所有的线程都停止运行。当用单步调试命令“step或next”，所有的线程开始执行。由于执行线程调度的是操作系统不是gdb，单步调试命令不能让所有的线程都单步。当前线程执行了一步，其他线程可能执行了N步。当执行next/step/continue时，当前线程完成单步运行前，其他线程运行遇到断点/信号/异常，gdb会选择一个遇到短信或信号中断的线程，切换线程时会提示“[Switching to Thread n]”

* set scheduler-locking
设置调度锁定模式，在一些系统中，gdb可以通过锁定操作系统线程调度，只允许一个线程运行。如果是on,单步调试命令会阻止其他线程抢占, 其他线程不会运行。如果是off，所有线程线程都会运行。当执行continue/util/finish 时，其他进程会恢复运行.

* show scheduler-locking
显示当前线程调度锁定状态

* set schedule-multiple
当执行continue/next/step时，gdb只允许当前进程下的线程恢复运行(fork出过个进程)。  
on: 所有进程下的线程恢复运行  
off: 当前进程下的线程恢复运行

* show schedule-multiple
显示多进程恢复模式

------
- [x] non-stop mode
在一些多线程的应用中，gdb支持只停止需要调试的线程，其他线程可运行不受影响。例如某些线程具有实时约束或必须继续响应外部事件，这是最小化的实时调试。称为不间断模式。
在non-stop mode中，当一个线程因为断点停止时，其他线程正常运行，continue/step 只适用于当前线程。
一般情况下在gdb启动或attach 一个进程时设置non-stop mode, 顺序执行如下命令，进入non-stop mode:
- Enable the async interface.
     set target-async 1

- If using the CLI, pagination breaks non-stop.
     set pagination off
     
- Finally, turn it on!
     set non-stop on
	
  continue -a, 让所有线程都继续执行, continue 只能让当前线程继续执行
  interrupt -a, 停止整个程序, interrupt/Ctrl-c 只能让当前线程挂起, 其他命令不支持-a.

------
- [ ] Background Execution
基本上用不到

------
- [ ] Thread-Specific Breakpoints
break linespec thread threadno
break linespec thread threadno if ..
threadno 从 info threads 中得到.
比如(gdb) break frik.c:13 thread 28 if bartab > lim  

------
- [ ] Interrupted System Calls
在使用gdb调试多线程程序时，有一个副作用。如果一个线程因断点或其他原因而停止，而另一个线程在系统调用中被阻塞，那么系统调用可能会提前返回。这是多线程和gdb用来实现断点和其他停止执行的事件的信号之间交互的结果。
例如： sleep (10); 如果不同的线程在断点处或出于其他原因停止，则调用sleep将提前返回。
```c++
       int unslept = 10;
       while (unslept > 0)
         unslept = sleep (unslept);
```  

  允许系统调用提前返回，因此系统仍然符合其规范。但是gdb确实会导致多线程程序的行为与没有gdb时不同。
  另外，gdb在线程库中使用内部断点来监视某些事件，例如线程创建和线程销毁。当这样的事件发生时，另一个线程中的系统调用可能会提前返回，即使您的程序似乎没有停止.  

------
- [ ] Observer Mode
略 


``` 
gdb sframe
// 设置non-stop 模式
set target-async 1
set pagination off
set non-stop on
// 设置断点
b ratms_base_rating.cpp:909 //insert
b ratms_base_sync.cpp:169  //truncate
// 启动
r -i xxxxx.xml
// 发送请求
ctrl+c
info threads
thread apply n continue
thread apply n continue

``` 


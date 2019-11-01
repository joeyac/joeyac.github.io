---
title: setrlimit 101
mathjax: true
categories:
  - Geek
tags:
  - Linux
date: 2018-11-05 17:27:23
---

在利用seccomp和ptrace对程序在系统调用行为上做出限制之后，剩下的就需要在资源上（如运行时间、使用内存）做出限制，这个需求可以使用`setrlimit`来实现。因此学习一下它的使用。

<!--more--> 
无论是`setrlimit`或者是`getrlimit`都是通过以下结构体进行资源定义：
```c
struct rlimit {
   rlim_t rlim_cur;  /* Soft limit */
   rlim_t rlim_max;  /* Hard limit (ceiling for rlim_cur) */
};
```
它们的的函数原型如下：
```c
int getrlimit(int resource, struct rlimit *rlim);
int setrlimit(int resource, const struct rlimit *rlim);
```
软限制是内核直接应用执行的资源量限制，硬限制则是作为软限制值的一个上界而存在。对于拥有`CAP_SYS_RESOURCE`权限的进程来说，可以任意调整软限制或者硬限制值，而对于非特权进程，只能在硬限制范围内调整软限制值，或者不可逆的降低硬限制值。

另外对于无限制，有一个特殊的值`RLIM_INFINITY`用来指示无限制，这个值在64位系统下应该为$2^{64}-1=18446744073709551615$

以下整理了一个常用的资源限制表：

|资源名|参数|单位|备注|
|:-:|:-:|:-:|-|
|地址空间|RLIMIT_AS|byte|即虚拟内存，向下取整到系统页大小，会影响`brk`,`mmap`,`mremap`等系统调用，一旦超出限制，程序会以`ENOMEM`错误退出。另外，如果无法通过`sigaltstack`自动扩容栈空间，将会生成`SIGSEGV`信号并终止程序。|
|转储文件大小|RLIMIT_CORE|byte|程序能够生成的转储文件大小。为0时不生成转储文件，大于0时将会截断多余的部分。|
|CPU时间|RLIMIT_CPU|s|限制进程能够消耗的CPU时间，如果达到了软限制，将会发送`SIGXCPU`信号，虽然这个信号默认行为是终止进程，但是是可以被重编程的。如果继续消耗CPU时间，达到硬限制时将会发送`SIGKILL`信号并强制终止程序。|
|数据段大小|RLIMIT_DATA|byte|限制程序的数据段大小，包括初始化数据，未初始化数据以及堆大小。取值将会向下取整到系统页大小，影响`brk`，`sbrk`，`mmap`等系统调用。一旦达到软限制将会抛出`ENOMEM`错误。|
|生成文件大小|RLIMIT_FSIZE|byte|进程能够创建的最大文件大小，超出限制时将会发送一个可重编程的`SIGXFSZ`信号，如果这个信号没有终止程序，相应的系统调用（如`write`，`truncate`）将会以`EFBIG`错误退出。|
|栈空间大小|RLIMIT_STACK|byte|程序所能使用的最大栈空间大小，一旦达到限制，将会生成`SIGSEGV`信号。|
|打开文件描述符数量|RLIMIT_NOFILE|-|限制程序最大能打开的文件描述符数量，影响`open`，`pipe`，`dup`等系统调用，如果超出限制，将会产生`EMFILE`错误。|
|进程数量|RLIMIT_NPROC|-|限制程序能够产生的最大进程数（在linux下，这个更精确的定义是线程数），一旦超出限制，`fork`系统调用将会以`EAGAIN`错误失败退出。注意，对于拥有`CAP_SYS_ADMIN`或者`CAP_SYS_RESOURCE`能力的进程来说这个限制无效。|

在上表中可以发现几个问题，首先涉及到内存的资源有`RLIMIT_AS`，`RLIMIT_DATA`，`RLIMIT_STACK`，这三个内存的关系在这里有比较直观的介绍：[memory-layout-of-c-program](https://www.geeksforgeeks.org/memory-layout-of-c-program/), 所以一般情况下直接对地址空间大小进行限制即可。

第二个是与时间相关的限制，可以看到只有一个CPU时间，并且其单位为秒，如果直接在Ubuntu终端使用time命令对程序执行进行测量，会发现三个时间：user cpu time、system cpu time和wall time。wall time如字面义，墙上时钟时间，即程序执行过程中实际流逝的时间。前两者则分别是用户程序和系统消耗的cpu时间。cpu time和wall time 两者是区分程序是否是并行程序的重要标准。对于传统的竞赛题目及代码来说，由于都是单核模型，wall time是一定不小于cpu time的。

回到上述限制本身，只能通过`setrlimit`限制其CPU时间，但是却能在程序结束时获得其较为精确的资源使用情况，假设我们程序要求是1.5s，直接设置CPU时间为2s甚至是3s，结束后再检查真实CPU运行时间即可。

当然，也需要对wall time做出限制，比如一个`while(true)sleep(100);`这样的程序基本上不会消耗什么CPU时间，却会一直占据资源，具体做法可以使用其他进程对执行程序进行监控并获取真实时间。

注意在linux系统下，有一些信号是无法被重编程的，如`SIGKILL`，`SIGSTOP`等等，在子进程退出时，父进程可以通过`wait`系统调用拿到子进程的退出状态码以及退出时收到的信号，据此判断退出状态。


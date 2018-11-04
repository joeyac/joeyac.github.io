---
title: ptrace 101
categories:
  - Geek
tags:
  - Linux
date: 2018-11-03 16:37:22
---

有一些其他的需求暂时没找到办法用seccomp实现，于是继上次探索seccomp之后，又开始了研究ptrace的使用。

<!--more-->

# 介绍

ptrace(2) - 系统调用 - process trace

使用ptrace可以暂停被监控程序，获取设置寄存器值和内存，监控系统调用，甚至是截断系统调用。一般用于程序debug，以及linux下的strace就是用ptrace实现的。

但是ptrace并没有被标准化，如果使用ptrace，需要关注自己的开发平台和系统架构，我的开发环境是Ubuntu 16.04 x86-64.
（我猜是因为不同系统架构或者平台寄存器都不一样？

函数原型如下：
```c
   #include <sys/ptrace.h>

   long ptrace(enum __ptrace_request request, pid_t pid,
               void *addr, void *data);

```

分为被跟踪程序（tracee）和跟踪程序（tracer），tracee只能被一个tracer跟踪，而tracer可以跟踪多个tracee.

一个非常经典的使用例子是fork出来一个子进程，然后调用`ptrace(PTRACE_TRACEME, 0, NULL, NULL)`表明这个子进程由其父进程追踪，父进程再使用相应的action code进行处理。

详尽的说明在[官方文档](http://manpages.ubuntu.com/manpages/xenial/man2/ptrace.2.html)可以看到。

我仅仅想在之前seccomp的基础上做出一些拓展，所以主要关注其中`PTRACE_O_TRACESECCOMP`相关的使用。

# 问题提出

先回到之前[seccomp-bpf](http://blog.jingwei.site/2018/10/31/seccomp%E7%9A%84%E4%BD%BF%E7%94%A8/#more)的例子上，假设我们默认允许所有系统调用,且仅仅允许输出到stdout, 一个测试的例子如下：
```c
int main() {
    // 不允许子进程获得新权限
    prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0);

    install_helper();

    scmp_filter_ctx ctx = NULL;

    // 默认允许所有系统调用
    ctx = seccomp_init(SCMP_ACT_ALLOW);

    // 只允许输出到stdout
    seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 1, SCMP_A0(SCMP_CMP_EQ, STDOUT_FILENO));
    seccomp_rule_add(ctx, SCMP_ACT_TRAP, SCMP_SYS(write), 1, SCMP_A0(SCMP_CMP_NE, STDOUT_FILENO));

    // 应用过滤器
    seccomp_load(ctx);

    // 释放内存
    seccomp_release(ctx);

    fprintf(stdout, "something to stdout\n");

    fprintf(stderr, "something to stderr\n");

    return 0;
}
```
能够正常产生下面的输出：
```bash
something to stdout
system call invalid: write(1): arg1=2
```
我想要让这个程序更通用，比如通过`execve`替换进程为其他任意可执行程序，另外简单编写了一个a.cpp文件并编译为名为`a`的可执行文件：
```
#include <bits/stdc++.h>
#include <unistd.h>
using namespace std;
int main() {
	fprintf(stdout, "something to stdout\n");
	fprintf(stderr, "something to stderr\n");
	return 0;
}
```
然后对原来的主程序做出一些修改：
```
void print_exit(int status)
{
    if (WIFEXITED(status))
        printf("normal termination, exit status = %d\n", WEXITSTATUS(status));
    else if (WIFSIGNALED(status))
        printf("abnormal termination, signal number = %d%s\n", WTERMSIG(status),
#ifdef WCOREDUMP
               WCOREDUMP(status) ? (" core file generated") : (""));
#else
        "");
#endif
    else if (WIFSTOPPED(status))
        printf("child stopped, signal number=%d\n", WSTOPSIG(status));
}

void child() {
    // 不允许子进程获得新权限
    prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0);

    install_helper();

    scmp_filter_ctx ctx = NULL;

    // 默认允许所有系统调用
    ctx = seccomp_init(SCMP_ACT_ALLOW);

    // 只允许输出到stdout
    seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 1, SCMP_A0(SCMP_CMP_EQ, STDOUT_FILENO));
    seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(write), 1, SCMP_A0(SCMP_CMP_NE, STDOUT_FILENO));

    // 应用过滤器
    seccomp_load(ctx);

    // 释放内存
    seccomp_release(ctx);

    // 用a.cpp替换子进程
    char cmd[100] = "{path_to_a}";
    char *argv[] = { "a", NULL };
    char *environ[] = { NULL };
    execve(cmd, argv, environ);
    puts("ERROR:");
    puts(strerror(errno));
    _exit(1);
}

int main() {

    pid_t pid = fork();

    if (pid < 0) _exit(1);
    else if (pid == 0) {
        child();
    } else {
        int status, ret;
        ret = wait(&status);
        printf("pid:%d, ret:%d, status=%d, %s\n", getpid(), ret, status, strerror(errno));
        print_exit(status);
    }
    return 0;
}
```
会看到类似下面的输出：
```bash
something to stdout
pid:11657, ret:11661, status=159, Success
abnormal termination, signal number = 31 core file generated
```
发现确实能够禁止非法的子程序的系统调用，这一点在seccomp的文档中就提到，所有的子进程都会继承父亲的seccomp设置，并且加上PR_SET_NO_NEW_PRIVS就可以确保子程序不能通过execve获得新权限，但是有一个问题，禁止的系统调用名并没有输出（即`install_helper`并没有继承到子进程当中

具体原因是，helper是通过注册信号处理函数来实现输出非法系统调用名的，在`sigaction`的[文档](http://man7.org/linux/man-pages/man2/sigaction.2.html)当中，有这样一个特殊说明：
> During an execve(2), the dispositions of handled signals are reset to the default; the dispositions of ignored signals are left unchanged.

也就是说在子进程中信号的handle被重置成默认值了……

也就是说不能考虑侵入子进程的方式来处理，如果从父进程接收信号呢？

很遗憾，经过测试，父进程也只会接收子进程退出时的SIGCHLD信号，并且是无法从这时记录的寄存器信息中拿到导致异常退出的系统调用信息的。

# 方案

于是我选择用ptrace！（还记得seccomp action当中的`SCMP_ACT_TRACE(x)`吗！

ptrace可以和seccomp配合使用，在某些系统调用处终止调用执行，并进行一些处理。

大致思路是，儿子进程非法系统调用都注册为`SCMP_ACT_TRACE(getppid())`，父进程通过循环不停的continue进程和处理相关的系统调用，然后通过ptrace获取子进程的用户空间的寄存器值，从中读取出来非法的系统调用和对应的参数。

首先增加下面三个头文件：
```
#include <sys/ptrace.h>
#include <sys/user.h> // 用户空间定义
#include <sys/reg.h>  // 寄存器定义

// 注意，网上很多资料针对的可能是老版本的linux内核，其在user.h中就包含了寄存器定义，
// 在较新的linux内核中要将上述两个头文件都包含进来
```
另外先注意一下原来helper函数当中，`REG_SYSCALL`和`REG_ARG0`两个宏定义（我的平台是x86_64架构：
```
#elif defined(__x86_64__)
#define REG_RESULT	REG_RAX
#define REG_SYSCALL	REG_RAX
#define REG_ARG0	REG_RDI
#define REG_ARG1	REG_RSI
#define REG_ARG2	REG_RDX
#define REG_ARG3	REG_R10
#define REG_ARG4	REG_R8
#define REG_ARG5	REG_R9
#endif
```
找到对应的寄存器：`REG_RAX`和`REG_RDI`，后面会用到。

main函数稍稍做出一些修改：
```
int main() {
    pid_t pid = fork();
    if (pid < 0) _exit(1);
    else if (pid == 0) {
        child();
    } else {
        int status;
        waitpid(pid, &status, 0);
        ptrace(PTRACE_SETOPTIONS, pid, 0, PTRACE_O_TRACESECCOMP);
        while (1) {
            if (wait_for_syscall(pid) != 0) break;
        }
    }
    return 0;
}
```
注意，在调用ptrace追踪之前，需要先调用一次wait（我现在还没太明白这里是为什么，希望万能网友解答……

然后增加了对子进程系统调用处理的函数`wait_for_syscall`，如果其返回值不为0，说明子进程已经结束。

首先child函数也做出了一些修改，在开头增加了一行`ptrace(PTRACE_TRACEME, 0, NULL, NULL);`表明这个子进程由其父进程进行追踪; 然后原来的禁止规则做出如下修改：
```
seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(write), 1, SCMP_A0(SCMP_CMP_NE, STDOUT_FILENO));
seccomp_rule_add(ctx, SCMP_ACT_TRACE(getppid()), SCMP_SYS(write), 1, SCMP_A0(SCMP_CMP_NE, STDOUT_FILENO));
```
表明这个系统调用规则将会由一个tracer进程追踪。

下面来看`wait_for_syscall`函数：
```
static int wait_for_syscall(pid_t child)
{
    int status;

    while (1) {
        ptrace(PTRACE_CONT, child, 0, 0);
        int ret = waitpid(child, &status, 0);

        printf("[waitpid status: 0x%08x]\n", status);
        printf("pid:%d, ret:%d, status=%d, %s\n", getpid(), ret, status, strerror(errno));
        print_exit(status);
        if (WIFEXITED(status) || WIFSIGNALED(status) ) {
            puts("exited");
            return 1;
        }
        // 判断是否是seccomp限制的规则，这个判断条件可以在ptrace文档中找到
        if (status >> 8 == (SIGTRAP | (PTRACE_EVENT_SECCOMP << 8))) {
            long syscall;
            syscall = ptrace(PTRACE_PEEKUSER, child, sizeof(long)*ORIG_RAX, 0);
//            long arg0;
//            arg0 = ptrace(PTRACE_PEEKUSER, child, sizeof(long)*RDI, 0);
            struct user_regs_struct regs;
            ptrace(PTRACE_GETREGS, child, NULL, &regs);
            printf("system call invalid: %s(%ld) with args: 0x%llx 0x%llx 0x%llx\n",
                   syscall < sizeof(syscall_names) ? syscall_names[syscall] : "null",
                   syscall,
                   regs.rdi, regs.rsi, regs.rdx);
            kill(child, SIGKILL);
            return 0;
        }
    }
}
```
注意在检查到规则之后，需要发送kill信号，且不能使用PTRACE_KILL: [reference1](https://lists.kernelnewbies.org/pipermail/kernelnewbies/2011-August/003132.html) [reference2](https://stackoverflow.com/questions/12015141/cancel-a-system-call-with-ptrace)

执行修改后的程序输出如下：
```
something to stdout
[waitpid status: 0x0007057f]
pid:21660, ret:21662, status=460159, Success
child stopped, signal number=5
system call invalid: write(1) with args: 0x2 0x400955 0x14
[waitpid status: 0x00000009]
pid:21660, ret:21662, status=9, No such process
abnormal termination, signal number = 9
exited
```

可以看到，基于ptrace这种机制，可以对系统调用做出丰富的限制，甚至可以直接修改函数的参数和返回值，结合seccomp能够完成十分强大的功能。


---
title: seccomp 101
categories:
  - Geek
tags:
  - Linux
date: 2018-10-31 22:12:19
---

最近有一些执行不安全程序的需求，考虑通过限制系统调用来对子程序进行行为限制（资源限制考虑用rlimit等方式，暂且不谈），学习了一下seccomp和seccomp-bpf的使用。

-- 纸上得来终觉浅，绝知此事要躬行


<!--more-->

# 介绍
seccomp是linux内核中一个基础的沙箱工具，在linux3.5之后开始支持seccomp-bpf扩展。BPF（Berkeley Packet Filter）是一种用于Unix内核网络数据包的过滤机制。只需要编写简单的过滤规则，就可以对程序系统调用进行可配置的限制，子进程会继承父进程所有的限制，并且可以配合linux的No New Privileges Flag保证避免类似execve的系统调用授予父级没有的权限。No New Privileges Flag使用也很简单，仅仅一行代码：`prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0);`

# 简单使用
我的开发环境是Ubuntu 16.04 amd64

首先安装对应的包：
`sudo apt install seccomp`

来一个简单的例子，假设我们想要限制程序只能向stdout写入数据：

```c
// main.c
// 省略了函数调用失败的处理

#include <stdio.h>
#include <unistd.h>
#include <seccomp.h>

int main() {
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

    fprintf(stdout, "something to stdout\n");

    fprintf(stderr, "someting to stderr\n");

    return 0;
}
```
简单解释一下一些关键字，`scmp_filter_ctx`是过滤器的结构体，需要通过`seccomp_init`或者`seccomp_rule_add`进行初始化规则或者添加规则，如果使用SCMP_ACT_ALLOW进行初始化，即为黑名单形式；若使用SCMP_ACT_KILL进行初始化，即为白名单形式。

```cpp
/**
 * Kill the process
 */
#define SCMP_ACT_KILL		0x00000000U
/**
 * Throw a SIGSYS signal
 */
#define SCMP_ACT_TRAP		0x00030000U
/**
 * Return the specified error code
 */
#define SCMP_ACT_ERRNO(x)	(0x00050000U | ((x) & 0x0000ffffU))
/**
 * Notify a tracing process with the specified value
 */
#define SCMP_ACT_TRACE(x)	(0x7ff00000U | ((x) & 0x0000ffffU))
/**
 * Allow the syscall to be executed after the action has been logged
 */
#define SCMP_ACT_LOG		0x7ffc0000U
/**
 * Allow the syscall to be executed
 */
#define SCMP_ACT_ALLOW		0x7fff0000U
```
以上这些seccomp actions当中，kill和allow的含义显而易见，分别是直接杀掉进程或者是允许进程执行。

SCMP_ACT_TRAP其实和kill类似，只不过kill是直接杀掉了进程并报invalid system call，而SCMP_ACT_TRAP会抛出一个能够捕捉的信号方便进行一些处理后再退出，这个下面会提到它的应用。

SCMP_ACT_ERRNO(x)指定一个触发时自定义的返回值。

SCMP_ACT_TRACE则是通知一个监控的ptrace进程进行一些处理。

SCMP_ACT_LOG字面意是在记录系统调用后再执行，但是我没太懂具体怎么记录的……希望有了解的朋友解惑。

下面介绍一下增加规则函数：
```c
int seccomp_rule_add(scmp_filter_ctx ctx,
		     uint32_t action, int syscall, unsigned int arg_cnt, ...);
```
ctx为scmp_filter_ctx结构体，action为上述scmp定义宏，syscall可以通过宏`SCMP_SYS({system_name})`拿到具体的id值，arg_cnt表明是否需要对对应系统调用的参数做出限制以及指示做出限制的个数，如果仅仅需要允许或者禁止所有某个系统调用，arg_cnt直接传入0即可；如果考虑到更高的自定义，需要先去了解一下具体系统调用的参数情况，然后再利用`SCMP_AX`及`SCMP_CMP_XX`类的宏定义做一些过滤。拿`read`调用举例子，先去http://man7.org/linux/man-pages/man2/read.2.html 查得其系统调用原型为：

```c
ssize_t read(int fd, void *buf, size_t count);
```

假设我们现在不允许从stdin读入count为100大小的数据块，则可以如下添加过滤器：
```c
seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(read), 2, 
				SCMP_A0(SCMP_CMP_EQ, STDIN_FILENO), 
                SCMP_A2(SCMP_CMP_EQ, 100))
```

最后，需要`seccomp_load(ctx)`应用过滤器，否则不会生效，以及使用`seccomp_release(ctx)`释放内存。

再回到之前编写的main.c文件，使用如下命令编译：`gcc main.c -lseccomp -o main`

执行得：
```bash
./main 
something to stdout
[1]    2548 invalid system call (core dumped)  ./main
```

将输出到stderr的那行注释掉后重新编译运行：

```bash
./main 
something to stdout
```

# 调试seccomp

虽然seccomp看起来很强大的样子，seccomp-bpf的使用方式似乎也非常简单，但是存在一个问题，调试起来不方便，在上面的例子当中，当使用了违规的系统调用时，仅仅报错，并没有额外的信息以供调试，在开发阶段非常不方便，至少能够输出违规调用的名称，才能方便调试。解决方案是通过使用sigaction注册对应的SIGSYS信号处理函数，所以之前的那行禁止规则需要改为`seccomp_rule_add(ctx, SCMP_ACT_TRAP, SCMP_SYS(write), 1, SCMP_A0(SCMP_CMP_NE, STDOUT_FILENO));`，即使用SCMP_ACT_TRAP抛出SIGSYS信号，然后捕捉并打印相应信息。

首先可以做一些准备，导出一个所有syscall的id和name对应头文件，打开终端，输入如下命令：
```bash
file=syscall-names.h
echo "static const char *syscall_names[] = {" > $file
echo "#include <sys/syscall.h>" | cpp -dM | grep '^#define __NR_' | LC_ALL=C sed -r -n -e 's/^\#define[ \t]+__NR_([a-z0-9_]+)[ \t]+([0-9]+)(.*)/ [\2] = "\1",/p' >> $file
echo "};" >> $file
```
会生成这样的syscall-names.h文件：
```c
static const char *syscall_names[] = {
 [247] = "waitid",
 [75] = "fdatasync",
 [245] = "mq_getsetattr",
 [204] = "sched_getaffinity",
 [42] = "connect",
 // ... 以下省略
```
然后在原来的`main.c`文件中添加三个函数以及所需头文件和宏定义：（这是从Linux kernel samples里撸过来的
```c
#define __USE_GNU 1
#define _GNU_SOURCE 1

#include <signal.h>
#include <sys/prctl.h>
#include <linux/types.h>
#include <linux/filter.h>
#include <linux/seccomp.h>
#include <linux/unistd.h>
#include <string.h>
#include <stddef.h>

#include "syscall-names.h"

#if defined(__i386__)
#define REG_RESULT	REG_EAX
#define REG_SYSCALL	REG_EAX
#define REG_ARG0	REG_EBX
#define REG_ARG1	REG_ECX
#define REG_ARG2	REG_EDX
#define REG_ARG3	REG_ESI
#define REG_ARG4	REG_EDI
#define REG_ARG5	REG_EBP
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

#ifndef SYS_SECCOMP
#define SYS_SECCOMP 1
#endif

const char * const msg = "system call invalid: ";

/* Since "sprintf" is technically not signal-safe, reimplement %d here. */
static void write_uint(char *buf, unsigned int val)
{
    int width = 0;
    unsigned int tens;

    if (val == 0) {
        strcpy(buf, "0");
        return;
    }
    for (tens = val; tens; tens /= 10)
        ++ width;
    buf[width] = '\0';
    for (tens = val; tens; tens /= 10)
        buf[--width] = (char) ('0' + (tens % 10));
}

static void helper(int nr, siginfo_t *info, void *void_context) {
    char buf[255];
    ucontext_t *ctx = (ucontext_t *)(void_context);
    unsigned int syscall;
    if (info->si_code != SYS_SECCOMP)
        return;
    if (!ctx)
        return;

    syscall = (unsigned int) ctx->uc_mcontext.gregs[REG_SYSCALL];
    strcpy(buf, msg);
    if (syscall < sizeof(syscall_names)) {
        strcat(buf, syscall_names[syscall]);
        strcat(buf, "(");
    }
    write_uint(buf + strlen(buf), syscall);
    if (syscall < sizeof(syscall_names))
        strcat(buf, ")");
    strcat(buf, "\n");
    write(STDOUT_FILENO, buf, strlen(buf));
    _exit(1);
}

static int install_helper() {
    struct sigaction act;
    sigset_t mask;
    memset(&act, 0, sizeof(act));
    sigemptyset(&mask);
    sigaddset(&mask, SIGSYS);

    act.sa_sigaction = &helper;
    act.sa_flags = SA_SIGINFO;
    if (sigaction(SIGSYS, &act, NULL) < 0) {
        perror("sigaction");
        return -1;
    }
    if (sigprocmask(SIG_UNBLOCK, &mask, NULL)) {
        perror("sigprocmask");
        return -1;
    }
    return 0;
}
```
然后在main函数中，加载过滤规则之前将对应的打印函数注册上：
```
    if (install_helper()) {
        printf("install helper failed");
        return 1;
    }
```
去掉stderr输出前的注释重新编译运行（记得带上syscall-names.h头文件）：
```bash
> gcc main.c syscall-names.h -lseccomp -o main
> ./main
something to stdout
system call invalid: write(1)
```
输出了非法的系统调用以及对应id，当然，也可以利用samples中的宏定义，输出其后的一些参数，但是参数类型有可能是char\* 等等，而寄存器当中的值也就仅仅是值而已，需要自己做一下强制转化再输出，由于参数类型不统一，好像没有什么统一的方式进行处理，或者直接按照long long int输出其值……`uc_mcontext.gregs`的关联定义如下：
```
/* Number of general registers.  */
#define NGREG	23

/* Container for all general registers.  */
typedef greg_t gregset_t[NGREG];

/* Context to describe whole processor state.  */
typedef struct
  {
    gregset_t gregs;
    /* Note that fpregs is a pointer.  */
    fpregset_t fpregs;
    __extension__ unsigned long long __reserved1 [8];
} mcontext_t;
```
在原来的基础上，增加输出第一个参数后，运行如下：
```bash
something to stdout
system call invalid: write(1): arg1=2
```
其中2是stderr的fileno，与预期相符。


参考资料：

[seccomp BPF](http://www.infradead.org/~mchehab/kernel_docs/userspace-api/seccomp_filter.html)

[no new privileges flag](http://www.infradead.org/~mchehab/kernel_docs/userspace-api/no_new_privs.html)

[linux kernel samples](https://github.com/torvalds/linux/blob/master/samples/seccomp/bpf-direct.c)

[teach seccomp](http://www.outflux.net/teach-seccomp/)

[teach seccomp autodetect](http://www.outflux.net/teach-seccomp/autodetect.html)

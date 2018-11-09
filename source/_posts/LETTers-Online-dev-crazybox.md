---
title: LETTers Online dev - crazybox
categories:
  - Geek
tags:
  - LETTers
date: 2018-10-19 12:55:22
---

LETTers Online中，crazybox的设计和开发记录。

-----Building-----

<!--more-->

经过调研，有以下几种方式完成sandbox部分的开发：

- 使用[QDUOJ](https://github.com/QingdaoU/OnlineJudge)后端的沙箱
- 使用[DMOJ](https://github.com/DMOJ/judge/)后端的沙箱
- 使用docker作为沙箱
- 使用ioi所用沙箱
- 完全自己实现沙箱和判题
	- 沙箱常见设计
		- ptrace (vsftpd)
		- seuid (chromium)
		- seccomp (chromium)

## 自己实现

参考资料：

- [chromium sandbox](https://chromium.googlesource.com/chromium/src/+/lkgr/docs/linux_sandboxing.md)

- [seccomp](https://veritas501.space/2018/05/05/seccomp%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)

linux3.5以后linux支持了Will Drewry的Seccomp-BPF的特性

主要有两个任务，一个是统计时间和内存，一个是限制非法的系统调用和限制时间以及内存

### 统计时间和内存

先明确一些概念：

#### 时间

- CPU time: 为CPU执行用户进程操作和内核系统调用所耗时间总和 (user+sys)
- real time / wall time: 实际流逝时间

#### 内存

- VSS- Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）
- RSS- Resident Set Size 实际使用物理内存（包含共享库占用的内存）

### 方案
- 方案1: 在python当中可以用psutil获取时间和max虚拟内存和max（不靠谱）
- 方案2: 利用/usr/bin/time获取时间和内存


## 一些记录

思路：首先完成纯c使用seccomp开发沙箱
[c libraries vs executable](https://stackoverflow.com/questions/33953732/shared-libraries-vs-executable)

然后将其包装成python函数进行使用

用seccomp-filter的方式开发沙箱，要确定哪些系统调用是合法的，可以通过strace -c command 查看一个命令的系统调用次数和时间占用。

使用setrlimit限制内存和时间 [介绍](https://www.cnblogs.com/niocai/archive/2012/04/01/2428128.html)

```
https://docs.python.org/3/extending/extending.html
https://stackoverflow.com/questions/11041299/python-h-no-such-file-or-directory
https://cmake.org/cmake/help/latest/module/FindPythonLibs.html
sudo apt-get install python3-dev  # for python3.x installs
```

在这两个链接中，分别有整个64位系统下的system call以及对应的分类:

[syscall_64.tbl](https://raw.githubusercontent.com/torvalds/linux/master/arch/x86/entry/syscalls/syscall_64.tbl)

[Classification and Grouping of Linux System Calls](http://seclab.cs.sunysb.edu/sekar/papers/syscallclassif.htm)

代码处理思路：
- 编译行为只需要利用setrlimit对资源做出限制即可（主要是编译时间和编译输出）
- 运行行为需要放入seccomp沙箱当中

### seccomp权限控制
- 允许任意的read调用
- 只允许向stdout和stderr的write调用
- 只允许列表中文件的的open调用
- 允许任意的close调用

打印可执行程序系统调用列表：
`strace -f -c ./act_samples 2>&1 | sed -n '8,$p' | awk '{print $NF}'`

交互题思路：

在python中调用os.pipe()和os.fork()，对两个程序分别调用cbox，利用两个管道分别重定向双方的stdin和stdout [link](https://www.tutorialspoint.com/python3/os_pipe.htm)


### 总体流程
首先主进程fork出一个从进程，从进程依次设置以下限制：

- `prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0);`
- `ptrace(PTRACE_TRACEME, 0, NULL, NULL);`
- 从配置文件当中读取时间内存（注意这里的时间是cpu时间）等限制，使用`setrlimit`做出限制
- `chdir`、`chroot`: `chroot`需要额外权限，可能不会使用
- 重定向`stdin`, `stdout`, `stderr`
- 增加seccomp规则
- 给自己发送`SIGSTOP`信号，这一步是因为，父进程需要通过`ptrace(PTRACE_SETOPTIONS, pid, 0, PTRACE_O_TRACESECCOMP);`来开始追踪子进程的非法系统调用，必须要在设置seccomp规则之后才能开始合法追踪，于是父进程可以先`wait`一下，如果获取到了子进程正常暂停，那么说明之前的限制都正确应用且此时可以开始追踪子进程。
- 调用`execve(config->file, config->argv, config->envp);`执行程序，注意在seccomp规则中需要额外增加一条允许`execve(config->file...)`这样的系统调用执行。另外注意调用execve之后，如果本身是被ptrace的进程，在执行成功execve之后，会发送一个`SIGTRAP`的信号。

主进程本身在fork之后，需要配合从进程进行一些处理：

- 首先调用一次`wait`，如果不是异常退出状态，说明限制正常应用，继续处理，否则退出程序。正常的信号应该为`SIGSTOP(18)`
- 设置ptrace追踪属性：`ptrace(PTRACE_SETOPTIONS, pid, 0, PTRACE_O_TRACESECCOMP);`
- 在子进程`execve`之前，用`setpgid(child,child);`将子进程的组id改为它自己，这样方便使用`kill(-child, XXX)`来向子进程及其派生进程发送信号。
- 调用`ptrace(PTRACE_CONT, child, 0, 0);`，然后子程序应该执行到execve完成之后
- 调用`wait`获取状态，信号应该为`SIGTRAP(5)`
- 调用`ptrace(PTRACE_CONT, child, 0, 0);`，从这里开始子程序应该开始真正的执行过程
- 调用`wait`获取状态，此时获取到状态时应该有三种情况:
	- 正常退出 / 资源异常
	- 使用了非法的系统调用
	- 超过wall time限制

在最后`正常退出 / 资源异常`放在一起是因为这种情况下都可以从wait返回的status中拿到大部分信息；`使用了非法的系统调用`需要特殊处理一下并且手动终止程序；`超过wall time限制`则是使用`setitimer`设置定时器向自身发送`SIGALRM`信号，并在对应的信号处理函数中将子进程组全部关闭。
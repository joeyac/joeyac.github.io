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


## 具体实现

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
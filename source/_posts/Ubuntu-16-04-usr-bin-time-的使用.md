---
title: Ubuntu 16.04 /usr/bin/time 的使用
categories:
  - Geek
tags:
  - Tools
  - Ubuntu
  - Resource
date: 2017-09-26 17:02:40
---

最近在尝试使用docker以及docker-py通过python实现一个基于Ubuntu 16.04运行代码的沙箱，之前遇到的问题暂且按下不表，最近遇到了一个很令人烦恼的问题。

Docker本身是有丰富的资源调度控制的，比如说使用的cpu核心数、cpu运行时间、实际运行时间、内存使用、交换内存使用等等，甚至可以利用ulimits来对linux系统进行限制，可以说是很完备了。

<!--more-->

获取时间以及stdout，stderr，exit code都很方便，但是发现，虽然可以限制最高内存，但是好像没有什么好方法获取运行期间达到的最高内存，Google、stackoverflow 搜了一圈都没找到什么好的解决方案，23333然后有一个不成熟的想法，既然可以限制最高内存，那么就可以通过二分实际使用最高内存……不过感觉这样搞很蠢……

然后突然想到自己好像有点思维僵化……一直在找docker获取内存最高使用的方法，为何不直接搜索linux是如何做的呢？然后就发现了这个[time has a verbose mode which gives you the maximum and average resident set size.](https://stackoverflow.com/questions/583779/how-can-i-determine-max-memory-usage-of-a-process-in-linux)

```bash
$ /usr/bin/time -v command_that_needs_to_measured |& grep resident
    Maximum resident set size (kbytes): 6596
    Average resident set size (kbytes): 0
```

ubuntu16.04中可能没有自带，需要使用`sudo apt install time` 安装，注意区分和系统自带函数time的区别，调用时需要带上路径。

然后尝试着测试了一下，首先是直接在代码里重定向标准输入输出，然后开了一个$10^7$大小的int数组且循环赋值（避免编译器优化），结果如下：

```bash
$ /usr/bin/time -v "./a" |& grep resident
	Maximum resident set size (kbytes): 40628
	Average resident set size (kbytes): 0
```

看起来很正常没有问题……然后我多了个心眼，测试了一下用`< >`来重定向输入输出，结果……

```bash
$ /usr/bin/time -v "./a < 1.in > 1.out" |& grep resident
	Maximum resident set size (kbytes): 1124
	Average resident set size (kbytes): 0
```

内存竟然变小这么多……不过也很好理解，显然是将“重定向”这个操作作为监控对象了，解决方案是加上`sh -c`参数以及在command之前加上`exec`，如下：

```bash
$ /usr/bin/time -v sh -c 'exec ./a < 1.in > 1.out' |& grep 'Maximum resident'
	Maximum resident set size (kbytes): 40632
```

至此问题完美解决。
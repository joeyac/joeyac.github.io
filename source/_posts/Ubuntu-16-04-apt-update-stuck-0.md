---
title: Ubuntu 16.04 Apt Update Stuck %0
date: 2017-09-10 21:26:43
categories:
  - Geek
tags:
  - Ubuntu
  - Ipv6
---

今天使用Ubuntu 16.04时，执行apt update时卡在了某个地方：

```bash
0% [Connecting to security.ubuntu.com (2001:67c:1360:8001::17)]
```
<!-- more -->
就是连接不上更新服务器，一个很奇怪的地方是后面的IP地址竟然是v6地址，若有所思的打开了校园网登录页面，看到了这样的界面：

![图片](http://ow2gecrwu.bkt.clouddn.com/Screenshot%20from%202017-09-10%2021-35-42.png)

哦也就是说之前更新使用了IPV6去更新，然后apt大概默认就以IPV6去更新List了？对于校园网偶尔获取不到V6地址，尝试了几次注销登录无果之后，找了一下apt强制走v4的方法：

```bash
sudo apt update -o Acquire::ForceIPv4=true
```

类似的，可以强制apt走v6流量，记录在此，以备不时之需。

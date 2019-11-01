---
title: install eclipse Neon on jdk9
mathjax: true
categories:
  - Geek
tags:
  - Eclipse
  - Java
  - Install
date: 2017-09-18 15:37:38
---

今天要写Java的时候突然发现还没有安装Eclipse，先是直接下载了网络安装包……结果因为墙的问题卡住……然后去下载完整安装包，直接运行却报错了：

```java
org.eclipse.e4.core.di.InjectionException: java.lang.NoClassDefFoundError: javax/annotation/PostConstruct
```

想了想看了一下Eclipse运行的配置文件，发现里面有这么几行：

```java
-Dosgi.requiredJavaVersion=1.8
```

然后看了下自己的java版本，竟然是jdk9……我啥时候装的java9……无奈不想卸载java9重装java8，求助于Google

## 解决方案

在$eclipse.ini$文件中，<code>-vmargs</code> 之下添加一行：
<code>--add-modules=java.se.ee</code>

## Reference
[Neon: how to run on jdk9?](https://stackoverflow.com/questions/34947994/neon-how-to-run-on-jdk9)
---
title: JudgeServer的设计和开发
tags:
  - OJ
id: 502
categories:
  - Geek
date: 2017-01-15 00:16:56
---

由现在的已有知识，可能要我完全设计一个判题沙箱是比较困难的，需要补充较多的操作系统的知识以及c底层的东西，感觉不太现实，时间成本太高，所以想由今年才了解的docker入手，以及加上python辅助来做JudgeServer。另外，为了方便后续添加多种语言，也需要考虑一种较为统一的格式来处理编译运行等操作。

那么，首先，

### docker是什么呢

Docker允许你很方便的在任何机器上部署web应用，并且不需要担心os和依赖库之类的东西，这是Docker设计的核心理念。

但是它远远不止这样，一个比较自然的想法就是，可以建立大量web应用甚至本地构建分布式集群环境，也可以在一个安全的环境下运行一些不值得信赖的代码。

不过需要注意的是，docker团队有提出，docker并不是绝对安全，可能存在绕过。

### 我考虑的策略

在github上寻求一个开源的运行代码的沙箱（isolate），然后将构建judgeServer配合isolate，将整个环境部署到Docker中去，再用真实机器的nginx将judgeServer代理出来，这样可以保证，首先，沙箱会杜绝大部分不安全的代码，其次，如果沙箱出了自己的内核错误，也只会影响到Docker这个虚拟机器，真实机器将Docker重新构建即可，方便快捷。

### 实现

首先安装好必备环境，isolate等等。。。

利用Python的commands（python2的commands.getstatusoutput，在Python3中被subprocess.check_output取代）库，直接调用安装好的isolate命令行集成环境，用一个class Runner 处理好其中的细节，比如内存开到限制内存的三倍，再方便监控是否是RE或者MLE等等，然后，再写一个class Compiler根据自定义的compile 配置文件向Runner扔参数和编译命令，同时注意捕获编译错误方便后续处理，同时，也需要一个class judger用来运行代码，重定向代码和判断PE，WA，AC等等，最后，利用方便的flask框架，根据需求构建API。

### 最终

以下所有api请求都需要添加如下header，并且以json类型传输数据：
<pre class="">headers= {"Authorization": 'Token ' + token,
          "Content-Type": "application/json"}</pre>
/ping/：用来测试JudgeServer的连接性以及查看JudgeServer的编译和运行参数；

/judge/：用来判题和返回判题结果

/sync/：用来同步单个或者所有题目用例

目前基本功能已经完成，但是代码整体上还是有很多可以优化的地方，所有代码都已经上传到[github](https://github.com/joeyac/JudgeServer)

&nbsp;
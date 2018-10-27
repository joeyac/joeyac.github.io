---
title: Online Judge&Virtual Judge 策划
tags:
  - OJ
id: 499
categories:
  - Geek
date: 2017-01-15 00:17:41
---

这个东西总是还想做，之前做的acmnote已经积攒了不少经验，发现了很多问题，有些甚至是要动整个结构的问题。。。于是觉得还是需要狠心切掉，重新做一个OJ，加上VJ，甚至考虑加上之前acmnote的功能，顺便学学计算机底层的一些东西。

&nbsp;

三大块概述：

&nbsp;

*   OJ
1.web部分：显然已经确定要用python以及django框架完成，关于前端，感觉vue很火的样子。。虽然很想用vue...但是感觉学的东西有点多，还是不要乱点技能树了，先用bootstrap搞搞吧，前端东西好多好乱啊= =
另外，需要学习一下restful的设计，转化成api访问。。。
2.judger部分：我觉得还是应该先实现judger，目前考虑用docker来监控运行情况
judger是核心部分，先考虑将这部分设计好，再看其他。

## Modules

<table class="modules" style="height: 444px;" width="950">
<tbody>
<tr>
<th>module</th>
<th>description</th>
<th>status</th>
</tr>
<tr>
<td>sandbox</td>
<td>Runs the contestant's solution in a controlled and secure environment, limiting its execution time, memory consumption and system calls. We have a stable implementation (`box`) based on ptrace and a new one (`isolate`) based on Linux kernel containers.</td>
<td class="statedone">works</td>
</tr>
<tr>
<td>judges</td>
<td>A set of utilities for comparing the solution's output with the correct answer at a given level of strictness.</td>
<td class="statedone">works</td>
</tr>
<tr>
<td>evaluator
(a.k.a. grader)</td>
<td>This module controls the whole process of grading the solution. It runs the compilers, the sandbox and the judges as described in configuration files.</td>
<td class="statedone">works</td>
</tr>
<tr>
<td>queue manager</td>
<td>Distributes grading between a cluster of computers, each of them running the evaluator.</td>
<td class="statepart">works, but needs revision</td>
</tr>
<tr>
<td>submitter</td>
<td>Handles submitting of solutions by contestants and passing them to the evaluation system. Contains a server daemon and a front-end for contestants. (If your contest uses a web-based contestant interface, you probably do not need this, although it can serve as a clean interface between your web services and the evaluator.)</td>
<td class="statepart">works, but needs revision</td>
</tr>
</tbody>
</table>

*   VJ
*   NOTE
2017年1月23日update:

目前完成：

Judge：

*   sandbox：经过多方考虑，主要是由于时间限制，sandbox采用ioi使用的isolate沙箱，日后若有时间再自己开发；
*   judgeserver：采用flask框架，构建restful api,基本构建完成，具体可以查看[JudgeServer](http://jingwei.site/judgeserver%e7%9a%84%e8%ae%be%e8%ae%a1%e5%92%8c%e5%bc%80%e5%8f%91/)
Web：在慎重考虑之后，还是决定采用django作为整体的构建框架，前端样式采用bootstrap和bootswatch以提供可主题化的样式，暂时利用jquery作为前端辅助（视学习vue.js的情况决定是否替代），在web这里，我认为没有必要完全构建成restful api的模式，只需要针对性的对某些内容进行序列化即可，django自带的model，form，view的模式还是挺高效的。

Web端分为这么几部分：

*   queuemanager：控制不同电脑上的集成的judgeServer，提供队列管理作用，将提交分发到最合适的JudgeServer，以及从JudgeServer获取信息和判题结果；
*   submiter：处理用户提交的代码，加入提交queue中；
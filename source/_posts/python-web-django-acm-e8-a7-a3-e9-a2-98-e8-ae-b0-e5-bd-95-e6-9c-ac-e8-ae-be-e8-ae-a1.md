---
title: Python-web django ACM解题记录本设计
tags:
  - django
  - python
  - web
id: 465
categories:
  - Geek
date: 2016-07-24 06:27:10
---

想要练习一下Python以及它的django框架，同时为自己的ACM训练提供一点便利。

首先需要导入三个主要OJ（POJ，HDOJ，codeforces）的题目，这些信息之前YZL的脚本可以拉取；

对于每个题目，定义一个记录（类似评论），每个用户可以为每个题目添加记录，记录是含有{Note，Tag}的一个模型，每个人对每个题目添加自己的理解，笔记和标签，同时可以方便统计各大oj大家做过的题目。

管理员可以添加比赛，比赛中的题目可以从现有题目列表里添加，也可以额外添加，额外添加的题目自动添加到题目列表中。
1.在某一场比赛内，生成每个人对应的总结（参考**LETTers 2016 Regional Training Report** Version 1.0），以及对应每个人和每个题的记录链接。
2.类比problem给contest增加记录{Note,Tag},contest中还需要有公告，以便提示Note所需要填写的内容

管理员对于每个题目可以添加一个官方性质的Tag，这个Tag在每个题目中默认隐藏显示。此Tag与Problem为多对多关系

其余对应关系：多个record对应一个Problem和一个user，多个Problem对应一个contest，每个总结对应一个user，多个总结对应一个contest。

&nbsp;

后期：   1.通过简单脚本拉取各个oj的题目列表；（是否需要？）

2.通过UserProfile里的各个oj的ID去拉取AC的题目，添加已做过题目标记。

首先建立一个site项目，其中应该含有account,problem,contest(Summary),record,utils应用；

account：正常用户登录，注册，验证app，其中model应有User以及Userprofile两项；

User：{username,real_name,email,create_time,admin_type,<span style="color: #ff0000;">problems_status</span>,

reset_password_token,reset_password_token_create_time,

auth_token,two_factor_auth,tfa_token,openapi_appkey,is_forbidden,}

UserProfile：{user,avatar,blog,hduoj_username,poj_username,

codeforces_username,<span style="color: #ff0000;">problems_status</span>,phone_number,school,student_id,}
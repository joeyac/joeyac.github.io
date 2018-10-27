---
title: gedit变身为编程利器的简单配置
id: 71
categories:
  - Geek
date: 2016-02-04 19:39:59
tags:
  - gedit
  - ubuntu
---

## 按照校队的要求，需要从java编程转型到c++编程以便队内交流，真是痛苦啊，为了少受干扰，于是转战至ubuntu系统，搭好了java和c，c++编程环境，却纠结在选择何种IDE，code:blocks，eclipse，vim...查找了许多资料，最终决定还是用gedit吧，比起其他的界面这个还是我比较能接受的哈哈哈，最主要还是功能基本能满足我的需求。下面给出简单配置：[转载出处](http://blog.csdn.net/u012965890/article/details/38472149)

操作系统：ubuntu 15.04

首先打开gedit，编辑-&gt;首选项，在查看、编辑器、字体和颜色这三个选项卡里选择自己喜欢的配置。比如缩进，代码高亮等。

用下面的命令来安装/更新gedit的插件：
<div class="dp-highlighter bg_plain">
<pre class="lang:default decode:true">sudo apt-get install gedit-plugins</pre>
然后在插件选项卡里选择自己所需的插件。我个人选择了以下插件：插入日期/时间、代码注释、单词补全、绘制空白、嵌入终端、括号补全、片段、拼写检查器、外部工具、文本大小、文档统计、文件浏览器面板。

</div>
片段（快速插入常用的文本片段）：

选择工具-&gt;Manage Snippet，可以对其进行管理，例如加入常用模板，以便加快coding速度。（就差块cherry青轴了&gt;_&lt;）

&nbsp;

&nbsp;

&nbsp;

![](http://img.blog.csdn.net/20140810191821773?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk2NTg5MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

嵌入终端（在底部面板中嵌入一个终端）：

默认快捷键是Ctrl+F9,也可以选择通过查看-&gt;底部面板把它调出来，这时候你会发现字体和背景很糟糕，请打开终端并输入以下命令：
<div class="dp-highlighter bg_plain">
<pre class="lang:default decode:true ">dconf-editor</pre>
选择org-&gt;gnome-&gt;gedit-&gt;plugins-&gt;terminal,在右边的面板中将"use-theme-colors"取消即可。

</div>
![](http://img.blog.csdn.net/20140810200023890?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk2NTg5MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

外部工具（执行外部命令和shell脚本）：

这个是神器！

选择工具-&gt;Manage External Tools，对其进行管理。

如果无法通过快捷键对程序进行编译运行，那以上的所有操作都是白费（如果你想每次都打开终端输入命令我也不反对），这个插件提供了很好的接口。

编译（以C/C++和Java为例）：

添加新工具，在右边的编辑栏中输入以下代码：
<div class="dp-highlighter bg_plain">
<pre class="lang:sh decode:true ">#!/bin/sh  

fullname=$GEDIT_CURRENT_DOCUMENT_NAME  

name=`echo $fullname | cut -d. -f1`  

suffix=`echo $fullname | cut -d. -f2`  

if [ $suffix = "c" ]; then  

    gcc $fullname -o $name -O2 -Wall -std=gnu99 -static -lm   

elif [ $suffix = "cpp" ] || [ $suffix = "c++" ] || [ $suffix = "cc" ] || [ $suffix = "cxx" ] || [ $suffix = "C" ]; then  

    g++ $fullname -o $name -O2 -Wall -std=gnu++0x -static -lm  

elif [ $suffix = "java" ];  then  

    javac $fullname -encoding UTF-8 -sourcepath . -d .  

fi</pre>
&nbsp;

</div>
编译选项的命令可以自己选择，设置成自己习惯的，以上编译选项部分参考了[ACM/ICPC的编译选项](http://icpc.baylor.edu/worldfinals/programming-environment)。然后设置自己习惯的快捷键，调整选项，以下是我的：

![](http://img.blog.csdn.net/20140810194940203?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk2NTg5MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

运行（以C/C++和Java为例）：

添加新工具，在右边的编辑栏中输入以下代码：

&nbsp;
<pre class="lang:sh decode:true ">#!/bin/sh  

fullname=$GEDIT_CURRENT_DOCUMENT_NAME  

name=`echo $fullname | cut -d. -f1`  

suffix=`echo $fullname | cut -d. -f2`  

dir=$GEDIT_CURRENT_DOCUMENT_DIR  

if [ $suffix = "c" ]; then  

    gnome-terminal --hide-menubar --working-directory=$dir -t "Terminal-$name" -x bash -c "$dir/$name; echo;echo 'press ENTER to continue';read"  

elif [ $suffix = "cpp" ] || [ $suffix = "c++" ] || [ $suffix = "cc" ] || [ $suffix = "cxx" ] || [ $suffix = "C" ]; then  

    gnome-terminal --hide-menubar --working-directory=$dir -t "Terminal-$name" -x bash -c "$dir/$name; echo;echo 'press ENTER to continue';read"  

elif [ $suffix = "java" ];  then  

    gnome-terminal --hide-menubar --working-directory=$dir -t "Terminal-$name" -x bash -c "java $name echo;echo 'press ENTER to continue';read"  

fi</pre>
![](http://img.blog.csdn.net/20140810194935015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk2NTg5MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
大概就是这么多了，更多的功能还要靠以后自己挖掘了。&gt;_&lt;

&nbsp;

Thanks for reading...&gt;_&lt;

&nbsp;
---
title: 在Ubuntu 14.04使用cron实现作业自动化
id: 163
categories:
  - Geek
date: 2016-03-04 17:39:26
tags:
---

##### Cron是Linux系统中最有用的工具之一，cron作业是在指定时间到来时被调度执行的作业。

最常见的自动化系统管理和自动维护工作，比如每天发出的按计划完成了备份的通知，或者是按计划定时清理/tmp/目录的通知。还有很多Web应用程序也需要执行定时作业。
本文讲述了Cron的工作机制，你可以用cron实现调度作业作业。Cron本身是一个守护进程，在后台运行，通过配置文件“crontab”来根据时间调度指定的作业执行。

## 一、启动Cron服务

基本上所有的Linux发行版在默认情况下都预安装了cron工具。即使未预装cron，也很简单，执行命令手动安装它：

    root@ubuntu-14:~# apt-get install cron `</pre>
    接着检查cron服务的状态，默认情况它应该运行于后台。如果它未启动，那么可以手动启动此服务。
    <pre>`root@ubuntu-14:~# service cron start
    root@ubuntu-14:~# service cron status 
    cron start/running, process 1027 `</pre>

    ## 二、使用Cron帮助

    如果cron工作正常，那么你可以使用man命令查看其手册描述的详细用法。
    <pre>`root@ubuntu-14:~# man crontab `</pre>
    上面的命令显示了crontab手册描述的使用方法。如果要查看怎样使用cron作业指定的信息，可以这样：
    <pre>`root@ubuntu-14:~# man 5 crontab `</pre>
    ![](http://blog.chinaunix.net/attachment/201508/4/301743_1438671703mgJE.png)
    要退出帮助命令手册的显示，按下q键或h键。
    ![](http://blog.chinaunix.net/attachment/201508/4/301743_1438671714o97X.png)

    ## 三、Crontab命令的用法

    下面讲述怎样使用crontab命令实现定时调度作业。

    ### 1、对Cron作业进行列表

    使用以下命令列出当前用户计划的cron作业。
    <pre>`root@ubuntu-14:~# crontab –l `</pre>
    会列出当前用户的所有cron作业，如果想查看其它用户的cron作业，可以使用如下命令：
    <pre>`root@ubuntu-14:~# crontab –l –u username `</pre>
    这会列出指定用户的cron作业。

    ### 2、编辑Cron作业

    要添加一个新cron作业，或者是编辑现有的cron作业，可以使用如下命令：
    <pre>`root@ubuntu-14:~# crontab -e `</pre>

    ### 3、移除Cron作业

    使用下面的命令移除已经计划的cron作业。
    <pre>`root@ubuntu-14:~# crontab –r `</pre>
    使用下面的命令移除所有已计划的cron作业，且无需再次确认。
    <pre>`root@ubuntu-14:~# crontab –ir `</pre>

    ### 4、命令参数

    -u user：用来设定某个用户的crontab服务；
    file：file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab。
    -e：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件。
    -l：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容。
    -r：从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件。
    -i：在删除用户的crontab文件时给确认提示。

    ## 四、用Crontab计划任务

    除了通过配置文件来处理计划cron作业之外，还有别的方法可以做到。如果你查看/etc目录，你会发现有这样的目录：cron.daily、 cron.hourly、cron.monthly等等。因此，把cron脚本放入这些目录中，那么系统会根据这些目录名定时执行这些作业脚本的。

    ### <a name="t10"></a>1、Cron配置类型

    Cron有两种配置文件类型，用于调度自动化任务。

    1）系统级Crontab
    这些cron作业被系统服务和关键作业所使用，且需要root级的权限才能执行。可以在/etc/crontab文件中查看系统级的cron作业。
    ![](http://blog.chinaunix.net/attachment/201508/4/301743_1438671736SsE2.png)
    2）用户级Crontab
    用户级的cron作业是针对每个用户单独分开的。因此每个用户都可以使用crontab命令创建自己的cron作业，还可以使用以下命令编辑或查看自己的cron作业。
    <pre>`root@ubuntu-14:~# crontab –e `</pre>
    ![](http://blog.chinaunix.net/attachment/201508/4/301743_1438671748xis6.png)
    选择编辑器后，你可以配置新cron作业了。

    ## 五、用Crontab调度作业

    可以使用指定的语法调度cron作业，而且还有速记缩写命令，使的管理cron作业很简单。
    Crontab语法如下：
    <pre>`* * * * * command to be executed
    - - - - - -
    | | | | | |
    | | | | | --- 预执行的命令
    | | | | ----- 表示星期0～7（其中星期天可以用0或7表示）
    | | | ------- 表示月份1～12
    | | --------- 表示日期1～31
    | ----------- 表示小时1～23（0表示0点）
    ------------- 表示分钟1～59 每分钟用*或者 */1表示 `</pre>

    ## 六、新Cron作业配置实例

    现在你已经熟悉了crontab命令、语法及cron作业的类型，现在可以创建一些作业计划进行测试。可以使用crontab –e 命令添加。

    ### 1、每分钟运行的计划作业

    下面的例子，创建一个cron作业，它每分钟输出文本“test cron job to execute every minute”并把文本发送到user@vexxhost.com邮箱。
    首先用crontab命令编辑
    <pre>`root@ubuntu-14:~# crontab –e `</pre>
    写入以下的脚本
    <pre>`SHELL=/bin/bash
    HOME=/
    MAILTO=”user@vexxhost.com”
    #This is a comment
    * * * * * echo 'test cron job to execute every minute'
    :wq!    保存并退出 `</pre>
    ![](http://blog.chinaunix.net/attachment/201508/4/301743_1438671765TMfD.png)
    一旦保存了此cron脚本文件，就可以把它添加到计划的作业中。

    ### 2、在指定时间调度Cron job作业

    假如想调度某个cron作业，让它在“每个星期四的下午7:00”运行，那么crontab脚本应该这样：
    <pre>`00 19 * * 4 sh /root/test.sh `</pre>
    再把它添加到调度作业中。
    <pre>`root@ubuntu-14:~# crontab -e
    crontab: installing new crontab 

上面脚本中的“00 19”指的是下午7点，“4”指的是星期四。

## 七、总结

可以看到，用crontab实现自动化任务是很容易的，而且它可以按分钟、小时、周、月、星期来执行任务。除此之外，Linux还有一个at命令，它适用于处理只执行一次的任务，且需要先运行atd服务。
其次要注意环境变量的问题。有时我们创建了一个crontab，但是这个任务却无法自动执行，而手动执行这个任务却没有问题，这种情况一般是由于在 crontab文件中没有配置环境变量引起的。在crontab文件中定义多个调度任务时，需要特别注环境变量的设置，因为我们手动执行某个任务时，是在 当前shell环境下进行的，程序当然能找到环境变量，而系统自动执行任务调度时，是不会加载任何环境变量的，因此，就需要在crontab文件中指定任 务运行所需的所有环境变量，这样，系统执行任务调度时就没有问题了。
还要注意清理系统用户的邮件日志。每条任务调度执行完毕，系统都会将任务输出信息通过电子邮件的形式发送给当前系统用户，这样日积月累，日志信息会非常大，可能会影响系统的正常运行，因此，将每条任务进行重定向处理非常重要。
最后要注意，新创建的cron作业，不会马上执行，至少要过2分钟才执行。如果重启cron服务则会马上执行。
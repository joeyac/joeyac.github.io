---
title: Ubuntu14.04配置msmtp和mutt
mathjax: true
id: 160
categories:
  - Geek
date: 2016-03-04 16:54:11
tags:
  - ubuntu
---

<div>

## 由于需要备份网站数据库，想要通过定时执行脚本发送打包好的数据库备份邮件到自己的邮箱，虽然说vps支持配置sendmail，还是太麻烦了，遂利用msmtp+mutt的形式用163邮箱（需开启SMTP服务，登录网页版163邮箱设置即可）

## 1.配置msmtp和mutt（sudo apt-get install安装即可）

</div>

### **1.1配置msmtp**

<div>创建msmtp日志文件“.msmtp.log”，在.msmtprc当中指定，注意这里的"."表示是隐藏文件，内容为空。</div>
<div>
<div class="cnblogs_code">
<pre class="">$ sudo vim ~/.msmtp.log</pre>
</div>
配置msmtp配置文件“.msmtprc”
<div class="cnblogs_code">
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"><a title="复制代码">![复制代码](http://common.cnblogs.com/images/copycode.gif)</a></span></div>
<pre class="">#Accounts will inherit settings from this section
defaults
# A first gmail address
account        gmail
host           smtp.gmail.com
port           587
from           username@gmail.com
user           username@gmail.com
password       password
tls_trust_file /etc/ssl/certs/ca-certificates.crt
# A second gmail address
account    gmail2 : gmail
from       username2@gmail.com
user       username2@gmail.com
password   password2
# A freemail service
account    freemail
host       smtp.freemail.example
from       joe_smith@freemail.example
user       joe.smith
password   secret
# A provider's service
account   provider
host      smtp.provider.example
# A 126 emali
account    126
host       smtp.126.com
port       25
from       aaa@126.com
auth       login
tls        off
user       aaa@126.com
password   password
logfile    ~/.msmtp.log
# Set a default account
account default : 126</pre>
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"><a title="复制代码">![复制代码](http://common.cnblogs.com/images/copycode.gif)</a></span></div>
</div>
配置.msmtprc权限，以下设置是只给.msmtprc的所属用户读和写的权限，其他人没有任何权限

</div>
<div>
<div class="cnblogs_code">
<pre>$ sudo chmod 600 .msmtprc --设置配置文件权限</pre>
</div>
如果要查看.msmtprc的所属用户，可以通过以下命令查看，我们可以看到，.msmtprc这个文件所属用户是root用户，组是root组。
<div class="cnblogs_code">
<pre>root@BJCGNMON01:~# ls -l .msmtprc 
-rw------- 1 root root 251 Feb 17 10:22 .msmtprc</pre>
</div>
以上设定很重要，使用什么账户去调用msmtp，那么该账户就要有对 .msmtprc文件的读写权限。

</div>

### **1.2配置mutt**

<div>mutt配置分为两种，看你是想全局生效还是某一单一用户生效。如果是系统全局设置，修改/etc/Muttrc这个配置文件；如果使用某个系统用户，可以需要修改“~/.muttrc”这个文件。</div>
<div>
<div class="cnblogs_code">
<pre class="">#sudo vim ~/.muttrc
set sendmail="/usr/bin/msmtp"
set use_from=yes
set realname="name"
set from=aaa@126.com
set envelope_from=yes</pre>
</div>
我只想给我当前root用户配置mutt功能，所以使用后者。修改完毕以后也需要查看这个文件的读写权限，当前是root账号要使用mutt功能，那么这个.muttrc就必须对于root账户有读写权限。查看权限的方法如下：
<div class="cnblogs_code">
<pre>root@BJCGNMON01:~# ls -l .muttrc 
-rw-r--r-- 1 root root 122 Feb 17 10:27 .muttrc</pre>
</div>

## **2.测试smtp的信息**

</div>

### **2.1msmtp测试**

测试命令：
<div>
<div class="cnblogs_code">
<pre>测试配置文件：msmtp -P
测试smtp服务器：msmtp -S</pre>
</div>
还有一种方法是在配置msmtp之前就可以进行测试，比如测试163的smtp的命令如下：

</div>
<div>
<div class="cnblogs_code">
<pre class="">bitnami@linux:~$ msmtp --host=smtp.163.com --serverinfo
SMTP server at smtp.163.com (smtp.163.gslb.netease.com [220.181.12.18]), port 25:
    163.com Anti-spam GT for Coremail System (163com[20121016])
Capabilities:
    PIPELINING:
        Support for command grouping for faster transmission
    STARTTLS:
        Support for TLS encryption via the STARTTLS command
    AUTH:
        Supported authentication methods:
        PLAIN LOGIN
This server might advertise more or other capabilities when TLS is active.</pre>
</div>
从返回信息中我们可以看到，这个smtp是支持TLS的，验证方式支持 PLAIN 和 LOGIN

</div>

### **2.2测试邮件**

<div>命令行输入：</div>
<div>
<div class="cnblogs_code">
<pre class="">echo "test" |mutt -s "my_first_test" aaa@126.com</pre>
</div>
如果是**多个收件人**，那么使用**空格或者逗号分开**即可，测试命令：
<div class="cnblogs_code">
<pre class="">echo "test" |mutt -s "my_first_test" aaa@126.com bbb@163.com
echo "test" |mutt -s "my_first_test" aaa@126.com,bbb@163.com</pre>
<pre class="">mutt -s "Title: mail test"  111111111@qq.com</pre>
<pre class="">mutt -s "Title: mail test" -a /home/test.tar.gz test@gmail.com &lt; /home/mailmessage.txt</pre>
-s后面是邮件的标题.

-a后面是发送的附件.

test@gmail.com 表示发送邮件的目的地址.

&lt; /home/mailmessage.txt 表示读取mailmessage.txt以邮件正文形式发送.

如果收到邮件就表示安装成功了，下一步配置备份脚本和定时任务。
<pre class="lang:sh decode:true">    #!/bin/bash
    #定义数据库的名字
    DataBakName=Data_$(date +"%Y%m%d").tar.gz
    WebBakName=Web_$(date +%Y%m%d).tar.gz
    #删除本地3天前的数据
    rm -rf /home/backup/Data_$(date -d -3day +"%Y%m%d").tar.gz
    rm -rf /home/backup/Web_$(date -d -3day +"%Y%m%d").tar.gz
    #导出mysql数据库
    /usr/local/mysql/bin/mysqldump -u root -ppassword dbname &gt; /home/backup/seeke.sql
    #压缩数据库
    tar zcf /home/backup/$DataBakName /home/backup/*.sql
    #删除sql文件
    rm -rf /home/backup/*.sql
    #压缩网站数据
    tar zcvf /home/backup/$WebBakName /home/wwwroot/
    延时10秒钟
    sleep 10s
    #备份数据到邮箱
    /usr/bin/mutt -s "Title:web+mysql backup" test@qq.com -a /home/backup/$DataBakName -a /home/backup/$WebBakName &lt;/home/backup/backup.txt
    #注意防火墙是否打开了25端口，要不然邮件发不出去.
    #mutt参数中 -s 指定发送邮件的标题
    #mutt参数中 -a 指定邮件包含的附件，如果多个附件，每个附件都需要一个 -a 参数
    #mutt参数中 &lt;/root/backup.txt 指定邮件的内容为backup.txt的内容</pre>
&nbsp;

</div>
</div>
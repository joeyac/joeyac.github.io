---
title: 创建自己的私人git服务器
id: 456
categories:
  - Geek
date: 2016-06-02 15:07:41
tags:
---

由于来回在windows还有linux下的双系统编程，源代码记录比较麻烦，一些，嗯，比较奇怪的代码又不想放在github上给别人看，遂自己研究了一波搭建简单的git服务器。方便代码在不同系统甚至不同电脑上转移和修改并且版本控制还是佷赞的。

&nbsp;

第一步，首先git应该是服务器必备的，如果没有安装，采用下面的命令：（ubuntu系统
<pre class="lang:sh decode:true ">apt-get install git</pre>
第二步，建议不要使用root账户来运行git服务，新建一个名为git的用户用来运行git服务：
<pre class="lang:c++ decode:true">sudo adduser git</pre>
第三步，创建证书登录：

收集所有需要登录的用户的公钥，就是他们自己的`id_rsa.pub`文件，把所有公钥导入到`/home/git/.ssh/authorized_keys`文件里，一行一个。

其中.ssh/及文件夹及authorized_keys建议用git用户创建。

在root用户下，可用下面命令创建：
<pre class="lang:sh decode:true ">mkdir /home/git/.ssh
chmod 700 /home/git/.ssh

touch /home/git/.ssh/authorized_keys
chmod 600 /home/git/.ssh/authorized_keys
chown git /home/git/.ssh/
chown git /home/git/.ssh/authorized_keys</pre>
可如下导入公钥：
<pre class="lang:sh decode:true ">cat &gt;&gt;.ssh/authorized_keys &lt;&lt;\EOF
paste-key-as-one-line
EOF
exit</pre>
另外，另外etc/ssh/sshd_config里面要修改几个参数

StrictModes no

RSAAuthentication yes

PubkeyAuthentication yes

AuthorizedKeysFile .ssh/authorized_keys

（均需要去掉前面的#，使其生效）

第四步，初始化Git仓库：

先选定一个目录作为Git仓库，假定是`/home/git/sample.git`，在`/home/git/`目录下输入命令：
<pre class="lang:sh decode:true ">sudo git init --bare sample.git</pre>
Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以`.git`结尾。然后，把owner改为`git`：
<pre class="lang:sh decode:true ">sudo chown -R git:git sample.git</pre>
第五步，禁用shell登录：

出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑`/etc/passwd`文件完成。找到类似下面的一行：
<pre class="lang:sh decode:true ">git:x:1001:1001:,,,:/home/git:/bin/bash</pre>
改为：
<pre class="lang:sh decode:true ">git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell</pre>
这样，`git`用户可以正常通过ssh使用git，但无法登录shell，因为我们为`git`用户指定的`git-shell`每次一登录就自动退出。

第六步，克隆远程仓库：

现在，可以通过`git clone`命令克隆远程仓库了，在各自的电脑上运行：
<pre class="lang:c++ decode:true "> git clone git@server:/home/git/sample.git</pre>
由于只是自己用，把自己的每个系统的公钥收集起来放到服务器的`/home/git/.ssh/authorized_keys`文件里就是可行的。如果团队有几百号人，就没法这么玩了，这时，可以用Gitosis来管理公钥。

&nbsp;
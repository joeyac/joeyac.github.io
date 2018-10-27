---
title: 解决wordpress xmlrpc.php利用漏洞及被攻击问题
tags:
  - hacker
id: 441
categories:
  - Geek
date: 2016-05-04 20:17:41
---

2016.05.03下午15.40左右，自己的wordpress博客突然登陆不上了，显示502 bad gateway错误（或者504 gateway timeout错误），很奇怪，遂ssh连上服务器，明显感觉到连接速度缓慢，由于这两天校园网正在升级，本以为是校园网的原因，遂又接入了自己多台他国服务器去尝试连接博客所在服务器，全部报错。

于是开始debug模式：

由于502或者504错误的原因可能有很多，只能一个个排除。

1.首先我的wordpress是建立在nginx+php5+mariadb基础上的，首先看是否nginx出现了故障，添加了一个conf配置文件，端口设为800，根目录下放置了一个“hello test”的index.html文件，通过jingwei.site:800可以正常访问，排除nginx的问题；

2.这时候突然发现空间有new mail，需要接收但是没有安装mailutils，apt安装时竟然提示内存不够，
free -m看一下内存768MB已使用720MB+，就算算上cached的占用，也是很不科学的。
于是继续排查，top命令查看内存具体使用情况，出现了大量php5-fpm且占用大量内存的进程，这个时候我还以为是php5又出现了什么bug（php5貌似确实很容易抽风），索性将nginx+php5+mariadb三个服务一次性重启，关闭时内存使用恢复到了正常情况，开启之后测试访问博客主页，大概几分钟才刷出来了主页，这时候再刷新博客又挂掉了。再次查看内存使用情况，又恢复到了原来的状态。

3.到这里的时候我就很是怀疑，哦不，应该说基本确定有人在攻击我的博客了，查看nginx的日志文件（位于/var/log/nginx/），cat error.log然后疯狂刷屏，对于我这样的个人博客，这样的访问简直太搞笑了，应该还是写了个脚本，强制终止cat输出，查看信息，不过基本就是这种形式：
<pre class="lang:sh decode:true">2016/05/03 05:02:16 [error] 1634#0: *566225 connect() to unix:///home/wordpress/phpfpm.sock failed (11: Resource temporarily unavailable) while connecting to upstream, client: 185.103.252.3, server: jingwei.site, request: "POST /xmlrpc.php HTTP/1.0", upstream: "fastcgi://unix:///home/wordpress/phpfpm.sock:", host: "45.32.249.139"</pre>
这个时候我注意到了“client: 185.103.252.3”，大致看了一下，基本请求都来自这么四个ip：

185.103.252.170俄罗斯
185.103.252.3俄罗斯
185.130.4.197多米尼加
185.130.4.120多米尼加

好嘛，就是有人在打我博客，而且应该就是一个人，只不过挂了多个vps或者挂了代理之类的，让我一直很迷惑的时，无冤无仇打我这么一个小型vps是要干啥？

//05.03结束，由于有课程和训练，只完成了这么多

//下面是05.04日的排查

第二天五四青年节，下午半天假，回来继续排查，再次调出log文件，发现了一个关键的地方：
<pre class="lang:sh decode:true">server: jingwei.site, request: "POST /xmlrpc.php HTTP/1.0",</pre>
上面四个ip的访问无一例外，都是在请求这个php文件，cd到wordpress主目录，果然有这样一个文件，而且很明显，对方是通过脚本的方式不断请求的，遂改名为xmlrpc.motherfuck.php,然后再重启数据库,nginx,phpfpm等服务，这时候能够正常访问博客了。

在重启的这段时间内，google了一下xmlrpc.php这个文件，原来这是wordpress自身存在的一个漏洞：
> Pingback 是三种类型的反向链接中的一种，当有人链接或者盗用作者文章时来通知作者的一种方法。可以让作者了解和跟踪文章被链接或被转载的情况。一些全球最受欢迎的 blog 系统比如 Movable Type、Serendipity、WordPress 和 Telligent Community 等等，都支持 Pingback 功能，使得可以当自己的文章被转载发布的时候能够得到通知。 WordPress 中有一个可以通过 xmlrpc.php 文件接入的 XMLRPC API，可以使用 pingback.ping 的方法加以利用。 其他 BLOG 网站向 WordPress 网站发出 pingback，当WordPress处理 pingback 时，会尝试解析源 URL。如果解析成功，将会向该源 URL 发送一个请求，并检查响应包中是否有本 WordPress 文章的链接。如果找到了这样一个链接，将在这个博客上发一个评论，告诉大家原始文章在自己的博客上。 黑客向使用WordPress论坛的网站发送数据包，带有被攻击目标的 URL（源 URL）。WordPress 论坛网站收到数据包后，通过 xmlrpc.php 文件调用 XMLRPC API,向被攻击目标 URL 发起验证请求。如果发出大量的请求，就会对目标 URL 形成 HTTP Flood。当然，单纯向 WordPress 论坛网站发出大量的请求，也会导致 WordPress 网站自身被攻瘫。 除了 DDoS 之外，黑客可以通过源 URL 主机存在与否将返回不同的错误信息这个线索，如果这些主机在内网中确实存在，攻击者可以进行内网主机扫描。
> 
> 
> [来源](http://yusi123.com/3764.html)
在google上转了一圈，这个漏洞大致有两种利用途径，一是用来爆破wp-admin后台账号密码，不过这个对我来说没有影响，因为我的账号密码都不是常规字典里能有的，而且，似乎这个利用方式已经被wordpress官方修复；其次，就是利用pingback的功能将wordpress所在服务器当做肉鸡，转而对第三个网站进行ddos攻击。所以就是被人当做肉鸡了=  =

另外从网上搜集了一下解决方案：

一开始确实考虑过用ufw直接屏蔽ip，但这样不能解决其他IP也出现这样的攻击，我们需要禁止xmlrpc文件的访问功能才可以彻底的解决。

我之前采用的改名其实和删除文件是一个道理，目的就是让xmlrpc无法访问；

另外还有三种方法：
> 第一种是屏蔽 XML-RPC (pingback) 的功能。
> 
> <div>
> 
> 
> add_filter(‘xmlrpc_enabled’, ‘__return_false’);
> 
> 
> </div>
> 
> 第二种方法就是通过.htaccess屏蔽xmlrpc.php文件的访问
> 
> <div>
> 
> <div>
> 
> 
> # protect xmlrpc
> 
> 
> &lt;Files xmlrpc.php&gt;
> 
> 
> Order Deny,Allow
> 
> 
> Deny from all
> 
> 
> &lt;/Files&gt;
> 
> 
> </div>
> 
> </div>
> 
> 第三种同样的是修改.htaccess文件，如果有用户访问xmlrpc.php文件，然后让其跳转到其他不存在或者存在的其他页面，降低自身网站的负担。
> 
> <div>
> 
> <div>
> 
> 
> # protect xmlrpc
> 
> 
> &lt;IfModule mod_alias.c&gt;
> 
> 
> Redirect 301 /xmlrpc.php http://example.com/custom-page.php
> 
> 
> &lt;/IfModule&gt;
> 
> 
> </div>
> 
> </div>
终于修复了=  =博客可以正常访问，然后又出现了后台访问速度极慢的问题：

google了一下，原因如下：
> 在新版的 WordPress 中，为了后台的美观度，开发者在页面上加入了 Google Web 字体，这本来会让英文显示更加精美。但在国内，由于 googleapis.com 等域名常年抽风(你懂的)，直接导致的结果就是后台经常打开一点就卡住打不开了，加载极为缓慢，其实我们只要移除 Google 在线字体即可恢复原来的速度。
05.03日为了测试网络，将火狐浏览器设为直连而谷歌是连的梯子， 所以才会出现这种情况。（之前都是默认整个系统翻墙的）

想想为了方便还是移除了google的在线字体，在主题顶部的function.php顶部加入：
<pre class="lang:php decode:true ">add_filter('gettext_with_context', 'disable_open_sans', 888, 4 );
function disable_open_sans( $translations, $text, $context, $domain )
{
if ( 'Open Sans font: on or off' == $context &amp;&amp; 'on' == $text ) {
$translations = 'off';
}
return $translations;
}</pre>
至此所有问题都完美解决。

&nbsp;

最后，对于怎么会被攻击，有一个小小的疑问，因为我自己的博客，除了一些熟人，基本是没有外人知道的，05.04由于一些原因在wx中发送了我博客某几篇博文的链接给别人，由于之前搞过一段时间wx公众号，如果用wx内置浏览器打开链接，它会将链接提交到服务器进行转换，再返回转换之后的链接地址，有点怀疑；另外就可能是，并不是针对我的网站攻击，而是对方利用某个关键字扫到我的网站，转而作为肉鸡去执行ddos。

╮(╯▽╰)╭还是自己太弱，努力学习！
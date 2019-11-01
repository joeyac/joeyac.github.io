---
title: 定制hexo博客
mathjax: true
date: 2017-09-05 02:01:47
categories:
  - Geek
tags:
  - hexo
  - 前端
---
之前花了点时间将博客从WordPress迁移到了hexo，并且使用了著名的next作为博客主题，整体上还是很不错的，然后自己稍微添加了一些东西，在这里做一些记录。
<!-- more -->
## Github
由于hexo可以将内容生成静态页面，那么就可以很方便的利用github来维护并利用username.github.io来运营博客，按照找到的资料，我将该github库新建了一个hexo分支，用来存储主要的代码，master分支用来push生成的静态页面。
主要参考了[CrazyMilk](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/#more)这篇博客内容，按照其中流程搭建之后，日常修改按照如下方式进行：

> 在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理：
依次执行<code>git add .</code>、<code>git commit -m “…”</code>、<code>git push origin hexo</code>指令将改动推送到GitHub（此时当前分支应为hexo）；
然后才执行<code>hexo generate -d</code>发布网站到master分支上。

## Git子模块
由于我将整个项目扔在github上维护，并且采用了Next主题，这时候问题就来了，我像定制一部分Next主题中的内容，但是又想保持上游的更新，这个时候Git子模块就派上用场了～

主要参考这个Git文档：[git工具 - 子模块](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

需要注意的是，如果现在已经有了对应的文件夹，需要将其舍弃之后重新拉取，然后每次就都可以本地将两个仓库独立使用git命令维护了。

## 一言网 && Next v5.1.2
在[LOJ](https://loj.ac/)的首页右上角，有一个小小的一言，我一直觉得这个东西挺不错的，然后右键审查元素，发现了一言网这个网站：[一言网](http://hitokoto.cn/)

### about

> 这个网站是干什么的？

> 一言网(Hitokoto.cn)创立于2016年，隶属于萌创Team，目前网站主要提供一句话服务。

> 动漫也好、小说也好、网络也好，不论在哪里，我们总会看到有那么一两个句子能穿透你的心。我们把这些句子汇聚起来，形成一言网络，以传递更多的感动。如果可以，我们希望我们没有停止服务的那一天。

> 简单来说，一言指的就是一句话，可以是动漫中的台词，也可以是网络上的各种小段子。
或是感动，或是开心，有或是单纯的回忆。来到这里，留下你所喜欢的那一句句话，与大家分享，这就是一言存在的目的。*
*:本段文本源自hitokoto.us.

> 我可以干什么呢？

> 您可以...
分享句子 : 注册并和大家分享感动你的那个句子。
获取接口 : 我们提供了Api（支持HTTPS）用以各位获取句子以及信息。
点赞 : 您可以为您喜欢的句子点赞。点赞越多，句子被取得到的几率越大。
获取感动 : 在茫茫句海中寻找能感动你的句子。只要刷新首页就好了。（不要忘记随手点赞）
More and more...

这里是官方的介绍，然后该网站提供了专门的API接口[http://hitokoto.cn/api](http://hitokoto.cn/api)

通过不同类型参数可以随机获取不同的一言，于是我写了一个小小的JS，在此之前我稍微研究了一下Next的目录结构，在Next项目的<code>layout</code>文件夹下，有一个<code>_custom</code>文件夹，在这里可以很方便的定制一部分内容，修改对应的swig文件即可。

然后我在<code>sidebar.swig</code>文件中添加了如下内容:
```html
<div class="hitokoto motion-element" id="hitokoto-loader">
	<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
	<script>
		$.get('https://sslapi.hitokoto.cn/?c=a', function (data) {
		data = JSON.parse(data);
		$('#hitokoto-content').css('display', '').text(data.hitokoto);
		if (data.from) {
		  $('#hitokoto-from').css('display', '').text('——' + data.from);
		}
		});
	</script>
	<div style="font-size: 1em;margin-top: 15px; line-height: 1.5em;" id="hitokoto-content"></div>
	<div style="text-align: right; margin-top: 15px; font-size: 0.9em; color: rgb(102, 102, 102);" id="hitokoto-from"></div>
</div>
```
最终效果即如博客侧边栏最下面所示。
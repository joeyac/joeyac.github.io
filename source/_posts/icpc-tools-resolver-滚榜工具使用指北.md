---
title: icpc-tools resolver 滚榜工具使用指北
categories:
  - Geek
tags:
  - Guide
date: 2018-05-22 21:11:16
---

网上ICPC 官方滚榜工具的资料并不多 = = 半年多之前给学校办校赛折腾了一会，虽然最终效果很成功，实际上还是有一些玄学问题没解决……另外整个流程其实是比较复杂的……特此记录一下

<!--more-->

- 第一个是中文支持，当时觉得可能是java内字符编码的问题，尝试了反编译jar文件， = =好像是做了混淆，而且时间也非常紧迫，就没有继续研究了……区域赛的时候应该都是有官方支持的，改起来可能友好很多，我们最后是将选手的拼音直接放了上去……
- 第二个问题是，启动的时候偶尔会卡在加载页面，具体表现为分辨率不是全屏或者icpc logo只出半屏之类的，可能是内存不够？当时也没有时间细究，解决方案是，如果卡住了就按esc退出，重新启动，只要开头正常显示，后面就能正常运行，于是我们先在后台将滚榜程序正常运行起来停在logo页面，然后把笔记本接上投影仪直接滚榜。

据我所看到的资料，以及icpc tools的wiki，不知道是他们喜欢折腾还是总是换维护人什么的 = =基本上每年工具所需要的数据格式都有大大小小的变动，当然，如果你用的是pc2那一套东西，他们都封装好了那是非常方便的，但是如果你是想从其他OJ系统导出榜单去喂数据，就很蛋疼了……

我去年11月份使用的版本是 `1.2.1416`，这个可以在lib文件下的VERSION文件看到。

百度云链接：[resolver.zip](https://pan.baidu.com/s/1uG3QgOzNVVFvonpLLTUJgQ)

我稍微分析了一下xml文件所必须的内容，整理了后续的各个关键项，但是不保证所有的都是必须的，只是按照我给出的项进行配置，肯定是能正常滚榜的。

我采用了CDP的方式给滚榜工具喂数据，主要是方便管理选手头像和照片。

文件目录结构如下：

- board/cdp/contest.json
- board/cdp/images/logo/
- board/cdp/images/team/
- board/lib/
- board/awards.sh
- board/resolver.sh

其中lib文件夹下为滚榜工具的库文件，awards.sh和resolver.sh是方便使用的脚本；contest.json是生成的比赛信息的json文件，images/logo/文件夹下的为选手头像（区域赛中应为队伍学校的校徽），images/team/文件夹下的为选手照片（区域赛中应为队伍合照），logo下的图片最好为png格式，team下的为jpg格式，都以选手（队伍）ID命名。

以下介绍具体流程：

1. 从OJ系统导出比赛的提交数据，按照给定格式生成xml文件；
2. 利用awards.sh脚本标记获奖类型和获奖选手，生成contest.json文件；
3. 复制对应的选手头像和照片，用生成的xml文件中对应的id重命名，然后放入logo和team文件夹；
4. 多次运行resolver.sh，直到开头icpc logo页面能够正常显示。

### 一、XML文件格式分析
由于大家可能使用的OJ也不同，我在这里就介绍一下滚榜所必须的xml文件内容以及对应的格式。

xml文件必须的内容，可以分为以下几个部分：

1. 比赛基本信息（比赛名称，持续时间之类的, info项）
2. 比赛区域信息（队伍所属大洲，如亚洲等，我在实际使用的时候以学院作为区域喂的数据, region项）
3. 判题结果信息（根据OJ的实际情况来，主要是设置各种判题结果怎么处理，比如说CE算不算罚时，每次错误提交算多少罚时之类的，judgement项）
4. 代码语言信息（随便加一加就好了，实际滚榜并不会用到，但是你需要保证你OJ的提交中的语言都在这里包含了，否则会报错， language项）
5. 队伍信息（选手信息， team项）
6. 题目信息（problem项）
7. 结果信息（对应所有的提交，按格式生成对应的结果信息，run项）
9. 增加比赛结束标志（finalized项）

PS: 生成xml文件我是用的python的ElementTree，使用起来还是很方便的～

首先所有的项都应该在一个`contest` dom内，sample如下：

```
<contest>
	<info>
    ...
    </info>
    
	<region>
    ...
    </region>
    
	<judgement>
    ...
    </judgement>
    
    <language>
    ...
    </language>
    
    <problem>
    ...
    </problem>
    
    <team>
    ...
    </team>
    
    <run>
    ...
    </run>
    
    <finalized>
    ...
    </finalized>
</contest>
```

其中除info项只能存在一个外，其他的都可以存在一个或多个。

接下来一项一项介绍……

首先是`info`项：

sample：
```
  <info>
    <length>4:00:00</length>
    <penalty>20</penalty>
    <started>False</started>
    <starttime>1512824400.0</starttime>
    <title>2017 USTB ACM-ICPC Final Contest</title>
    <short-title>2017 USTB ACM-ICPC Final Contest</short-title>
    <scoreboard-freeze-length>0:30:00</scoreboard-freeze-length>
    <contest-id>default--3</contest-id>
  </info>
```
`info`内的各项不用多介绍了吧，看名字也都知道是干啥的，`contest-id`随便填就行，主要是注意一下`scoreboard-freeze-length`，`starttime`和`length`这三个参数，在sample中，第一个代表的是最后半小时封榜，`starttime`你需要把比赛开始的时间转化为时间戳形式，如果使用python，可以这样：
```
    starttime = ET.SubElement(info, 'starttime')
    starttime.text = str(time.mktime(timezone.localtime(contest.start_time).timetuple()))
```

注意`info`项有且只有一个。

接下来是`region`项：

这一项是在world final用来标记不同大洲的参赛队伍，以便于显示`亚洲第一`这样的奖项，我在实际使用中直接设为了学院名称。`external-id`自己随便分配一下就好了，但是要注意，后面还会用到。

sample：

```
<region>
  <external-id>1</external-id>
  <name>School of Computer and Communication Engineering</name>
</region>
<region>
  <external-id>2</external-id>
  <name>School of Automation and Electrical Engineering</name>
</region>
```

接下来是`judgement`项：

作用是为了配合`run`项使用，设置不同的结果以及对应的类型，比如可以设置CE不算罚时之类的，注意这里需要能和你自己OJ的提交信息完全匹配上，或者你也可以只设置AC和WA两种状态，然后导出提交的时候处理一下映射。

sample：

```
  <judgement>
    <id>1</id>
    <acronym>AC</acronym>
    <name>Yes</name>
    <solved>true</solved>
    <penalty>false</penalty>
  </judgement>
  <judgement>
    <id>2</id>
    <acronym>WA</acronym>
    <name>No - Wrong Answer</name>
    <solved>false</solved>
    <penalty>true</penalty>
  </judgement>
  <judgement>
    <id>3</id>
    <acronym>CE</acronym>
    <name>No - Compile Error</name>
    <solved>false</solved>
    <penalty>true</penalty>
  </judgement>
  <judgement>
    <id>4</id>
    <acronym>RE</acronym>
    <name>No - Run Time Error</name>
    <solved>false</solved>
    <penalty>true</penalty>
  </judgement>
  <judgement>
    <id>5</id>
    <acronym>SE</acronym>
    <name>No - System Error</name>
    <solved>false</solved>
    <penalty>false</penalty>
  </judgement> 
```

接下来是`language`项：

和`judgement`一样，主要是为了配合`run`使用。

sample：

```
  <language>
    <id>1</id>
    <name>c</name>
  </language>
  <language>
    <id>2</id>
    <name>c++</name>
  </language>
  <language>
    <id>3</id>
    <name>java</name>
  </language> 
```

接下来是`team`项：

为了方便，`id`和`external-id`我是直接用的数据库中用户的主键id

sample：

```
  <team>
    <id>87</id>
    <external-id>87</external-id>
    <region>School of Computer and Communication Engineering</region>
    <name>Yuncheng Wang</name>
    <university>Yuncheng Wang</university>
  </team>  
  <team>
    <id>108</id>
    <external-id>108</external-id>
    <region>School of Computer and Communication Engineering</region>
    <name>Jinkai Xue</name>
    <university>Jinkai Xue</university>
  </team> 
```

接下来是`problem`项：

sample：

```
  <problem>
    <id>8</id>
    <letter>H</letter>
    <name>Stones</name>
  </problem>
  <problem>
    <id>9</id>
    <letter>I</letter>
    <name>Practice of SUOAO</name>
  </problem> 
```

接下来是`run`项：

`id`我是导出了所有提交之后，从1开始计数的，没有尝试过不从1开始会不会出现问题……

`judged`需要全部为True，`status`需要全部为done，`timestamp`类似上面比赛的开始时间，将提交时间转化为时间戳；

`time`为从比赛开始到提交所经过的秒数，可以使用如下代码：

```
    times = ET.SubElement(run, 'time')
    times.text = str((item.create_time - contest.start_time).total_seconds())
```

sample：
```
  <run>
    <id>6</id>
    <judged>True</judged>
    <language>c++</language>
    <problem>2</problem>
    <status>done</status>
    <team>69</team>
    <time>471.309999</time>
    <timestamp>1512824871.31</timestamp>
    <solved>false</solved>
    <penalty>true</penalty>
    <result>WA</result>
  </run>
  <run>
    <id>7</id>
    <judged>True</judged>
    <language>c</language>
    <problem>4</problem>
    <status>done</status>
    <team>99</team>
    <time>494.562765</time>
    <timestamp>1512824894.56</timestamp>
    <solved>false</solved>
    <penalty>true</penalty>
    <result>WA</result>
  </run> 
```

接下来是`finalized`项：

注意这一项是必须的……否则后面某个步骤会出问题……


sample：

`time`直接设为0即可

`timestamp`是比赛结束时间的时间戳

`last_gold,last_silver,last_bronze`这三个指的是金牌、银牌、铜牌的最后一名是多少，比如sample中就代表金牌一个，银牌三个，铜牌五个

```
  <finalized>
    <last_gold>1</last_gold>
    <last_silver>4</last_silver>
    <last_bronze>9</last_bronze>
    <time>0</time>
    <timestamp>1512838800.0</timestamp>
  </finalized> 
```

注意上述所有项都需要放在一个`contest`的大项内部。

### 二、生成contest.json文件

终端输入命令`./awards.sh`

会弹出一个GUI界面, 然后选择第三项，`read local event feed`，并选择上一步生成的xml文件

然后就是GUI操作设置奖项了……一共有五种类型的awards可以设置，其中rank是设置前多少名才会显示大屏幕的暂停页面，就是滚榜的时候会不会停下来显示选手的照片，medal是金银铜牌，Group是区域第一名之类的，在我使用的时候就是设置每个学院的第一名，然后first to solve是一血，world finals直接忽略就好了……

设置完成之后点击save，会保存一个json文件，重命名为`contest.json`，放入`cdp`文件夹内

注：打星不发牌之类的，直接在导出的`contest.json`中修改对应的awards即可

### 三、拷贝并重命名需要的图片

现在`cdp`文件夹内仅有一个`cdp/contest.json`文件，在`cdp`文件夹下创建一个`images`文件夹，`images`下再创建`logo`文件夹和`team`文件夹，以之前每个选手在yml文件中分配的id给对应的图片命名即可。

如果有选手不存在头像或者照片也没关系，会默认显示一个icpc的logo或者照片。

### 四、运行resolver.sh

直接运行：`./resolver.sh cdp/`即可

注意我开头提到的玄学问题……加载不成功就esc退出重新加载就好了……

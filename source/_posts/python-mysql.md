---
title: Python & Mysql
tags:
  - python
id: 538
categories:
  - Geek
date: 2017-03-05 18:16:56
---

为了准备GPLT比赛，需要自己拼凑散题和练真题，散题的话，直接vjudge就好，真题的话，需要到源oj上去，不方便以比赛形式练习，自己开发的oj功能也不够完善，时间紧迫，经过和队长smile的商讨，最后决定采用爬虫+hustoj的形式解决需求。

我负责爬虫部分，操作hustoj的数据库，然后模拟登录GPLT官网提交代码，拉取结果，然后再更新数据库，至于hustoj前端的修改，全部交由了队长处理。

下面主要记录一下思路和碰到的坑，其实大多坑都是编码问题，也学到了很多：

1.交题：这个在之前开发OJ的时候，就抽象了一套比较完善的爬虫框架，结合GPLT练习题系统，最终选定了使用python的robobrowser框架，再配合beautifulsoup进行一些处理，可以很方便的进行模拟登录，交题，获取结果等操作，但是在这里需要注意的一点是，这里得到的结果之后需要插入数据库，而robobrowser默认是放回的Unicode的字符串，需要进行encode，像这样：
<pre class="lang:python decode:true ">cols = [ele.text.strip() for ele in cols]
case_result.append([ele.encode('utf8') for ele in cols])</pre>
我们需要将case_result也存到数据库里，前端取用，为了尽量仿真，采取的做法是如上构建数组，然后调用json.dumps(case_result)之后写入数据库，可是出现了一些问题，因为编码的问题，中文存储进去都变成了'\xE7\xAD\x94\xE6\xA1\x88...'这样的形式，从网上找到了这样的内容：
> 我们知道，python中的字符串分普通字符串和unicode字符串，**一般从数据库中读取的字符串会自动被转换为unicode字符串**
> 
> 
> **下面回到重点，使用json.dumps时，一般的用法为：**
> 
> 
> &gt;&gt;&gt; obj={"name":"测试"}
> 
> 
> &gt;&gt;&gt; json.dumps(obj)
> 
> '{"name": "[\\u6d4b\\u8bd5"}'](file://u6d4b//u8bd5%22%7D)
> 
> 
> &gt;&gt;&gt; print json.dumps(obj)
> 
> {"name": "\u6d4b\u8bd5"}
> 
> 
> &gt;&gt;&gt; json.dumps(obj).encode("utf-8")
> 
> '{"name": "[\\u6d4b\\u8bd5"}'](file://u6d4b//u8bd5%22%7D)
> 
> 
> 可以看到这里输出的字符串为普通字符串，但是里面的内容却是unicode字符串的内容，即使对结果进行encode("utf-8") ，因为这个字符串本身就已经编码过了，所有进行encode不会有变化
> 
> 
> &nbsp;
> 
> 
> 要想得到字符串的真实表示，需要用到参数**ensure_ascii=False(默认为True)**：
> 
> 
> &gt;&gt;&gt; json.dumps(obj,ensure_ascii=False)
> 
> '{"name": "\xe6\xb5\x8b\xe8\xaf\x95"}'
> 
> 
> &gt;&gt; print json.dumps(obj,ensure_ascii=False)
> 
> {"name": "测试"}
OK添加了之后确实可以，结果数据库又报错Incorrect string value: 很快查出了问题，也暴露了自己基础不扎实：在向数据库添加表的时候，忘记设置表的编码为utf-8，导致无法识别，于是drop掉表重新建立，问题解决。

关于数据库的操作，其实就是几个sql语句而已，写成函数方便调用即可。

另外一个重点是考虑到要满足并发的需求，考虑使用多线程或者多进程，在这点上python都可以很方便的处理，注意到我们的爬虫其实没有什么计算，属于io密集型，采用多线程更为合适，另外比赛人数也不算多，不需要增加额外的队列使用分布式之类的东西，直接用自带的threading和Queue即可满足要求。

&nbsp;

但是GPLT官网上其实是有提交频率的限制的，我的想法是从json配置文件里读取对应的账号密码，生成对应个数的多线程worker，worker类：
<pre class="lang:python decode:true ">class SubmitWorker(Thread):
    def __init__(self, uid, pwd, queue):
        self.uid = uid
        self.pwd = pwd
        self.time_stamp = 0
        self.queue = queue
        self.db = MySQLdb.connect("localhost", "test", "test", "oj", charset='utf8')
        super(SubmitWorker, self).__init__()

    def run(self):
        while True:
            sid = self.queue.get()

            cur_time = time.time()

            if cur_time - self.time_stamp &lt; 15:
                wait_time = int(15+self.time_stamp-cur_time+1)
                logger.info('{uid} should wait for {wait}s.'.format(uid=self.uid, wait=wait_time))
                time.sleep(wait_time)

            res = submit(self.db, sid, self.uid, self.pwd)

            if not res:
                s = '{uid} submit {sid} failed.'.format(uid=self.uid, sid=sid)
                logger.exception(s)

                self.time_stamp = time.time()

                self.queue.task_done()

            else:
                result = res['result']
                score = int(result[2])

                status = str(result[1].encode("utf-8"))
                try:
                    status = RESULT_MAP[status]
                except:
                    status = RESULT_MAP['default']

                time_s = int(result[5]) if result[5] else 0
                memory_s = int(result[6]) if result[6] else 0
                case_result = json.dumps(res['case_result'], ensure_ascii=False)
                # db, solution_id, score, result_id, time_s, memory_s, case_result
                update(self.db, sid, score, status, time_s, memory_s, case_result)

                self.time_stamp = time.time()

                self.queue.task_done()</pre>
添加了时间戳，以便节省不必要的网络开销。

主函数类似这样初始化worker：
<pre class="lang:python decode:true">    info = json.load(open('user-pwd.json'))
    for ele in info:
        uid = ele['user']
        pwd = ele['pwd']
        worker = SubmitWorker(uid, pwd, que)
        worker.daemon = True
        worker.start()

    .......
    que.join()</pre>
大概就是这样，有什么记起来的，再更新上来。
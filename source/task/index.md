---
title: 任务
date: 2017-10-30 18:40:35
---

<script>
    (function(){
        if("letters" !== prompt("请输入密码"))
        {
            alert("密码错误！");
            history.back();
        }
    })();
</script>

# CODERFORCES RUSH
Codeforces Round #443 (Div. 2) 

Codeforces Round #421 (Div. 2) 2017.11.07 night 20:00

# 2017 USTB 校赛

### 竞赛平台选择

- ~~计蒜客~~ (不支持封榜)
- hustoj (老牌oj，饱经考验，界面略丑)
- [x] qduoj （新生oj，可能会出锅，界面友好, 有封榜，搭建简单）
- loj （新生oj，可能出锅，界面友好，无封榜，搭建简单）
- [x] pc^2 (配置复杂，并且不支持直接预览题面，但是完美支持需求)
- [ ] dmoj (比赛得分模式复杂，没来得及研究)

最后暂定使用qduoj，hustoj备用，qduoj beta测试 pending...

qduoj webserver容器：
export LC_ALL=zh_CN.UTF-8
export LANG=zh_CN.UTF-8
解决中文乱码
docker exec -it 5e7aa5a8dd52 /bin/bash

docker inspect ab95fdc47b4c

```python
python manage.py shell < myscript.py
```

or

```python
./manage.py shell
>>> execfile('myscript.py')
```

# TODO
- [ ] 数电实验报告
- [ ] 微机接口-汇编语言学习
- [ ] 操作系统作业（三次，周二早第一节课交）
- [ ] 数电作业(八次，周三？周五？交)
- [ ] 熟悉mathlab


- [x] 加登录验证码
- [x] 批量导入账号
- [x] 测试比赛功能（主要看打星）
- [x] 讨论版功能
- [ ] 导入账号并且添加小组
- [ ] 发气球
- [ ] 滚榜


```
python3 -m pip install -U --force-reinstall pip
python -m pip install -U --force-reinstall pip
```

will leave you with a pip pointing to python2, a pip2 pointing to python2, and a pip3 pointing to python3.


批量发送邮件：http://www.jianshu.com/p/010a71ec11d6

ss客户端配置

http://www.itwendao.com/article/detail/318079.html

    "9070":"ustbletters",
    "9080":"ustbletters",
    "9090":"ustbletters",
    "9004":"ustb8834",
    "9001":"ustb1108",
    "9099":"blog.jingwei.site"
    
    
## 讨论版功能添加：

#### Require:

1. 查看post list
2. 发表post
3. 查看post detail
4. 发表reply
5. 通知

#### View:
 - contest_posts_list_page(request, contest_id, page=1)
 - post_replys_list_page(request, post_id, page=1)
 
 - PostAPIView(APIView):
 	- post(self, request): 发表post，单向通知管理员
 - ReplyAPIView(APIView):
 	- post(self, request): 发表reply，检查是否public通知
 - Notification(APIView):
 	- get(self, request): check是否有新提醒，通过查询NotificationMiddle确定是否存在对应的中间件，不存在就创建，然后查询[存储时间，当前时间]段内的notification是否存在，更新存储时间，返回结果，配合ajax周期用

#### LOGIC:
有任何一个人发表post则通知管理员，回复默认不通知，除非是管理员回复并且勾选全局通知

#### HTML:
1. 比赛详情 题目列表 提交 排名 讨论区

```python
r'^contest/(?P<contest_id>\d+)/posts/$'
r'^contest/(?P<contest_id>\d+)/post/(?P<contest_post_id>\d+)/$'
r'^contest/(?P<contest_id>\d+)/reply/(?P<contest_reply_id>\d+)/$'
```
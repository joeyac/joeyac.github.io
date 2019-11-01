---
title: poj3190 Stall Reservations解题报告
tags:
  - 优先队列
  - 日常怀疑人生系列
  - 贪心
id: 239
categories:
  - ACM
date: 2016-03-11 18:51:05
---

链接:[poj3190](http://poj.org/problem?id=3190)
> <div class="ptt" lang="en-US">Stall Reservations</div>
> 
> <div class="plm">
> 
> <table align="center">
> 
> <tbody>
> 
> <tr>
> 
> <td>**Time Limit:** 1000MS</td>
> 
> <td colspan="3">**Memory Limit:** 65536K</td>
> 
> </tr>
> 
> <tr>
> 
> <td>**Total Submissions:** 4735</td>
> 
> <td>**Accepted:** 1688</td>
> 
> <td>Special Judge</td>
> 
> </tr>
> 
> </tbody>
> 
> </table>
> 
> </div>
> 
> 
> Description
> 
> <div class="ptx" lang="en-US">Oh those picky N (1 &lt;= N &lt;= 50,000) cows! They are so picky that each one will only be milked over some precise time interval A..B (1 &lt;= A &lt;= B &lt;= 1,000,000), which includes both times A and B. Obviously, FJ must create a reservation system to determine which stall each cow can be assigned for her milking time. Of course, no cow will share such a private moment with other cows.
> 
> 
> Help FJ by determining:
> 
> 
> *   The minimum number of stalls required in the barn so that each cow can have her private milking period
> *   An assignment of cows to these stalls over time
> 
> Many answers are correct for each test dataset; a program will grade your answer.</div>
> 
> 
> Input
> 
> <div class="ptx" lang="en-US">Line 1: A single integer, N
> 
> 
> Lines 2..N+1: Line i+1 describes cow i's milking interval with two space-separated integers.</div>
> 
> 
> Output
> 
> <div class="ptx" lang="en-US">Line 1: The minimum number of stalls the barn must have.
> 
> 
> Lines 2..N+1: Line i+1 describes the stall to which cow i will be assigned for her milking period.</div>
> 
> 
> Sample Input
> 
> <pre class="sio">5
> 
> 1 10
> 
> 2 4
> 
> 3 6
> 
> 5 8
> 
> 4 7</pre>
> 
> 
> Sample Output
> 
> <pre class="sio">4
> 
> 1
> 
> 2
> 
> 3
> 
> 2
> 
> 4</pre>
> 
> 
> Hint
> 
> <div class="ptx" lang="en-US">Explanation of the sample:
> 
> 
> Here's a graphical schedule for this output:
> 
> <pre class="">Time     1  2  3  4  5  6  7  8  9 10
> 
> 
> Stall 1 c1&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;
> 
> 
> Stall 2 .. c2&gt;&gt;&gt;&gt;&gt;&gt; c4&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; .. ..
> 
> 
> Stall 3 .. .. c3&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; .. .. .. ..
> 
> 
> Stall 4 .. .. .. c5&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; .. .. ..</pre>
> 
> Other outputs using the same number of stalls are possible.</div>
<div class="ptx" lang="en-US"></div>
<div class="ptx" lang="en-US">大意就是有多个牛要挤奶,但是每个牛都特别犟,只在自己的时间内产奶,且不与其他牛共享stall,可以添加任意个stall,求最小的stall数且输出每个牛隶属的stall..</div>
各种坑....不多说,对时间限制相当严格,直接贪心于是就直接爆时间了,

其实还是对堆的应用不熟练啊,

正确思路应该是首先建立一个结构体存储每个牛的产奶时间的左右端点,然后按照产奶开始时间从小到大排序,优先处理先开始产奶的奶牛,同时结构体内还需要一个变量记录每个牛的id,以便排序之后的处理和输出:
<pre class="lang:c++ decode:true ">struct Cow
{
	int l,r,co;
}c[maxn];
bool cmp(Cow c1,Cow c2){
	if(c1.l==c2.l)return c1.r&lt;c2.r;
	else return c1.l&lt;c2.l;
}</pre>
然后,建立一个表示stall的优先队列,这个优先队列每次先取出队列中最早结束的奶牛结构体Cow,

比如现在优先队列中有m头奶牛,对第m+1头奶牛,如果这头奶牛的开始时间小于等于优先队列中最早结束产奶的奶牛的结束时间,那么就需要增加一个stall,于是将这头奶牛加入优先队列;如果这头奶牛的开始时间大于上述时间,那么取出优先队列的top,并将这头奶牛加入队列,同时维护一个ans值和ansl数组以便输出.

对优先队列,应该首先对结构体的&lt;运算符进行重载:
<pre class="lang:c++ decode:true">bool operator &lt;(const Cow&amp; a,const Cow&amp; b) 
{
	if(a.r==b.r)return a.l&gt;b.l;
	else return a.r&gt;b.r;
}</pre>
下面是完整AC代码:<span id="transmark"></span>
<pre class="lang:c++ decode:true ">/*
* Filename:    poj3190d.cpp
* Desciption:  我要疯了,真的
* Created:     2016年03月11日 14时16分29秒 星期五
* Author:      JIngwei Xu [mail:xu_jingwei@outlook.com]
*
*/
#include&lt;bitset&gt;
#include&lt;iostream&gt;
#include&lt;stdio.h&gt;
#include&lt;algorithm&gt;
#include&lt;cstring&gt;
#include&lt;cstdio&gt;
#include&lt;cmath&gt;
#include&lt;math.h&gt;
#include&lt;queue&gt;
#define INT_MAX 1&lt;&lt;30
using namespace std;
typedef long long ll;
const int INF=0x7F;
int n;
const int maxn=50000+7;
int anl[maxn];
struct Cow
{
	int l,r,co;
}c[maxn];
bool cmp(Cow c1,Cow c2){
	if(c1.l==c2.l)return c1.r&lt;c2.r;
	else return c1.l&lt;c2.l;
}
bool operator &lt;(const Cow&amp; a,const Cow&amp; b) 
{
	if(a.r==b.r)return a.l&gt;b.l;
	else return a.r&gt;b.r;
}
Cow c1;
priority_queue&lt;Cow&gt; pq;
int sel=1,ans=1;
void solve(){
	pq.push(c[0]);
	anl[c[0].co]=ans;
	for (int i = 1; i &lt; n; i += 1)
	{
		c1=pq.top();
		if (c[i].l&gt;c1.r)
		{
			anl[c[i].co]=anl[c1.co];
			pq.pop();
			pq.push(c[i]);
		}else{
			ans++;
			anl[c[i].co]=ans;
			pq.push(c[i]);
		}
	}
	printf("%d\n",ans);
	for (int i = 0; i &lt; n; i += 1)
	{
		printf("%d\n",anl[i]);
	}
}

int main()
{
//ios_base::sync_with_stdio(0);
#ifdef JIngwei_Xu
	freopen("data.in","r",stdin);
	freopen("data.out","w",stdout);
#endif
	while (scanf("%d",&amp;n)!=EOF)
	{
		for (int i = 0; i &lt; n; i += 1)
		{
			scanf("%d%d",&amp;c[i].l,&amp;c[i].r);
			c[i].co=i;
//			pq.push(c[i]);
		}
		sort(c,c+n,cmp);
//		for (int i = 0; i &lt; n; i += 1)
//		{
//			cout&lt;&lt;c[i].co&lt;&lt;":"&lt;&lt;c[i].l&lt;&lt;","&lt;&lt;c[i].r&lt;&lt;endl;
//		}
//		while (!pq.empty())
//		{
//			cout&lt;&lt;pq.top().co&lt;&lt;":"&lt;&lt;pq.top().l&lt;&lt;","&lt;&lt;pq.top().r&lt;&lt;endl;
//			pq.pop();
//		}
		solve();
	}
	return 0;
}</pre>
&nbsp;